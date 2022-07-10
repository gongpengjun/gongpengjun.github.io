---
layout: post
title: InnoDB表空间布局理解
date:   2022-07-09 09:00:00
categories: mysql
---

MySQL中最常用的存储引擎是InnoDB，存储引擎最核心的任务是把数据存放在磁盘上，本文尝试理解InnoDB文件在磁盘上的逻辑布局和物理布局。

- - -

## 1、页 page

磁盘是块存储设备，为了最大化磁盘的访问速度，InnoDB按照页为最小单位来读写硬盘，按最常见配置，一个页是16KiB。

## 2、区 extent

### 2.1、extent 和 xdes entry

一个extent是(页号)连续的64个页，extent的元数据是extent descriptor(简称xdes entry)，一个xdes entry管理一个extent的64个页。

一个页是16KiB，所以一个extent大小是1MiB (16KiBx64=1024KiB=1MiB)。

<img src="https://gongpengjun.com/imgs/innodb/innodb_extent_xdes_entry.svg" width="100%" alt="extent and xdes entry">

一个xdes entry (代码为`struct xdes_t` )占用40Byte，这些xdes entry存放在哪儿呢？

### 2.2、xdes page 和 xdes entry 和 extent

InnoDB用专门的页来存储xdes entry，这种页叫做xdes page，一个xdes page可以存256个xdes entry（即管理256个extent）。

一个extent的大小是1MiB，所以一个xdes page管理256MiB的文件存储空间。

<img src="https://gongpengjun.com/imgs/innodb/innodb_extent_page.svg" width="100%" alt="xdes entry">

## 3、段 segment

### 3.0、segment 和 inode

大神Jeremy Cole在[Page management in InnoDB space files](https://blog.jcole.us/2013/01/04/page-management-in-innodb-space-files/)中File segments and inodes 小节中描述了 segment和inode之间的关系：

原文：

> File segments and inodes are perhaps where InnoDB terminology and documentation are murkiest. an INODE entry in InnoDB merely describes a file segment (frequently called an FSEG).

译文:

> 文件segment和inode可能是InnoDB术语和文档最晦涩的地方。InnoDB中的INODE条目仅仅描述了一个文件段(通常称为FSEG)。

这是说：在InnoDB语境下，segment和inode表达同一个意思，可以互换使用。

在InnoDB源码中主要使用`inode`，在文档中在主要使用`segment`。

说说我的感受：一说起inode，我需要想一下到底是在说什么，而一说起segment，我就立刻知道在说一个索引的叶子段、非叶子段或Undo段，和InnoDB的核心议题直接关联起来了。所以，为了(我)便于理解，下文主要使用segment，把inode都替换成了segment。

具体就是：

segment entry <=等价于=> inode entry

segment page <=等价于=> inode page

Tips：看源代码、官方文档、或者书籍、网上的博客文章，尝试进行上面的等价变换，就容易理解了。

### 3.1、segment 和 segment entry

segment是一个逻辑上的概念，segment管理的空间也不要求连续。一个segment包括32个碎片页和三个extent的链表（FREE exent链表、NOT FULL extent链表、FULL extent链表）。segment的元数据是segment entry（实际代码中是inode entry）。

<img src="https://gongpengjun.com/imgs/innodb/innodb_segment_inode_entry.svg" width="100%" alt="xdes page">

一个segment entry占用192Byte，这些segment entry存放在哪儿呢？

### 3.2、segment page 和 segment entry 和 segment

InnoDB用专门的页来存储segment entry，这种页叫做segment page，一个segment page可以存85个segment entry（即管理85个segment）。

<img src="https://gongpengjun.com/imgs/innodb/innodb_segment_inode_page.svg" width="100%" alt="segment page">

## 4、索引index和段segment

InnoDB中数据就是索引，索引就是数据。一个聚簇索引就是一颗B+树，一个二级索引也是一颗B+树。

为了提高访问速度，每颗B+树的叶子节点页放在Leaf Node Segment中管理，内部(非叶)子节点页放在Internal Node Segment中管理。

每颗B+树的根节点页里面包含这两个segment的segment entry的位置（指针），这个信息就是FSEG Header。

<img src="https://gongpengjun.com/imgs/innodb/FSEG_Header.png" width="100%" alt="FSEG Header">

## 5、InnoDB表空间布局全貌

来看看Jeremy Cole在[The basics of InnoDB space file layout](https://blog.jcole.us/2013/01/03/the-basics-of-innodb-space-file-layout/)一文 Space files 小节中画的space file概览图：

<img src="https://gongpengjun.com/imgs/innodb/Space_File_Overview.png" width="100%" alt="Space File Overview">

按照我的术语，理解如下图：

<img src="https://gongpengjun.com/imgs/innodb/Space_File_Overview_with_comments.png" width="100%" alt="Space File Overview with Comments">

一个ibd文件可能几十上百GB，甚至几TB，最大可以达到64TiB。InnoDB将这个大文件逻辑上划分为每256MiB一组，每组的第一个页是专门存放xdes entry的XDES Page，正好可以管理本组256MiB的空间。其中ibd文件的第一个页(页号为0)还包含tablespace的元信息，所以这个xdes page也叫做File Space(FSP_HDR) Page。



关于[tablespaces vs filespace](https://blog.jcole.us/2013/01/03/the-basics-of-innodb-space-file-layout/) 概念的说明：

原文：

> InnoDB’s data storage model uses “spaces”, often called “tablespaces” in the context of MySQL, and sometimes called “file spaces” in InnoDB itself.

译文：

> InnoDB 的数据存储模型使用“空间”，在 MySQL 的上下文中通常称为“表空间”，有时在 InnoDB 本身中称为“文件空间”。

## 6、索引index的物理布局全貌

再来看MySQL官方博客引用的Jeremy Cole在[Page management in InnoDB space files](https://blog.jcole.us/2013/01/04/page-management-in-innodb-space-files/)一文最后画的一个InnoDB索引文件的全景图：

<img src="https://gongpengjun.com/imgs/innodb/jeremycole_innodb_segment_and_extent.jpeg" width="100%" alt="Index File Segment Structure">

按照我的术语，理解如下图：

<img src="https://gongpengjun.com/imgs/innodb/jeremycole_innodb_segment_and_extent_comment.jpeg" width="100%" alt="Index File Segment Structure with Comments">

其中：

- Page 3 (INDEX) 是B+树的根节点页面，里面存有指向Leaf Segment Entry和Internal Segment Entry的FSEG Header结构。
- Page 2(INODE) 是Segment Page，专门存放segment entry，所以FSEG Header里的指针指向Page 2里的segment entry
  -  segment entry即inode entry（图中简写为inode），这里inode是个非常晦涩的用法，在图中其实就是segment entry的含义。

- 每个segment entry里面包含着32个碎片页（Frag Array）和三个xdes链表（Free  List、Not Full  List、Full List），
  - 碎片页数组Frag Array直接指向大小为16KiB的页面
  - 三个xdes链表中链接的是大小为40Byte的xdes entry（每个xdes entry管理一个连续64个页的extent）
- 每256MiB的第一个页面是专门存放xdes entry的xdes page
  - Page 0：第一个xdes page因为包含管理tablespace的元信息，所以该页叫FSP_HDR
  - Page 16384：即第二个256MiB中的第一个页是纯粹存放xdes entry的xdes page
    - 页号16384即16KiB，说明该页前面有16KiB*16KiB=256MiB的空间

## 7、表空间tablespace的逻辑布局全貌

看懂了IBD文件的物理布局和索引Index的物理布局，再来看经常见到的表空间tablespace的逻辑布局，心中就了然了。

<img src="https://gongpengjun.com/imgs/innodb/innodb_tablespace_logic_layout.jpeg" width="100%" alt="Tablespace Logic Layout">

这是一个由具象到抽象的提炼总结过程，提炼出的东西自然心中有数。我更擅长先具象，再抽象，因为这样没有疑问和悬念，心里踏实。

## 8、参考

- [InnoDB : Tablespace Space Management](https://dev.mysql.com/blog-archive/innodb-tablespace-space-management/)
- [Extent Descriptor Page of InnoDB](https://dev.mysql.com/blog-archive/extent-descriptor-page-of-innodb/)
- [Page management in InnoDB space files](https://blog.jcole.us/2013/01/04/page-management-in-innodb-space-files/)
- [The basics of InnoDB space file layout](https://blog.jcole.us/2013/01/03/the-basics-of-innodb-space-file-layout/)
- [Data Organization in InnoDB](https://planet.mysql.com/entry/?id=36803)
- [Data Organization in InnoDB.pdf](http://www.royalwzy.com/wp-content/uploads/2015/11/Data-Organization-in-InnoDB.pdf)
- [『译』InnoDB中的数据组织](http://pollilog.cn/2017/10/31/innodb-data-organization/)
