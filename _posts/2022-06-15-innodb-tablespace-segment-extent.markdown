---
layout: post
title: InnoDB表空间布局理解
date:   2022-06-15 09:00:00
categories: MQ
---

MySQL中最常用的存储引擎是InnoDB，而InnoDB最核心的任务是把数据存放在磁盘上，我们一起看看InnoDB文件在磁盘上的逻辑和物理布局。

- - -

## 1、区 extent

### 1.1、extent 和 xdes entry

一个extent是(页号)连续的64个页，extent的元数据是extent descriptor(简称xdes entry)，一个xdes entry管理一个extent的64个页。

一个页是16KiB，所以一个extent大小是1MiB (16KiBx64=1024KiB=1MiB)。

<img src="https://gongpengjun.com/imgs/innodb/innodb_extent_xdes_entry.svg" width="100%" alt="extent and xdes entry">

一个xdes entry (代码为`struct xdes_t` )占用40Byte，这些xdes entry存放在哪儿呢？

### 1.2、xdes page 和 xdes entry 和 extent

InnoDB用专门的页来存储xdes entry，这种页叫做xdes page，一个xdes page可以存256个xdes entry（即管理256个extent）。

一个extent的大小是1MiB，所以一个xdes page管理256MiB的文件存储空间。

<img src="https://gongpengjun.com/imgs/innodb/innodb_extent_page.svg" width="100%" alt="xdes page">

## 2、段 segment

### 2.0、segment 和 inode

大神Jeremy Cole[博客文章](https://blog.jcole.us/2013/01/04/page-management-in-innodb-space-files/) File segments and inodes 小节中描述了 segment和inode之间的关系：

原文：

> File segments and inodes are perhaps where InnoDB terminology and documentation are murkiest. an INODE entry in InnoDB merely describes a file segment (frequently called an FSEG).

译文:

> 文件segment和inode可能是InnoDB术语和文档最晦涩的地方。InnoDB中的INODE条目仅仅描述了一个文件段(通常称为FSEG)。

用大白话就是说：在InnoDB语境下，segment和inode表达同一个意思，可以互换使用。

我的理解：

在InnoDB源码中主要使用`inode`，在文档中在主要使用`segment`。

说起inode需要想一下到底是什么，而segment则比较直观，为了便于理解，下文主要使用segment，把inode都替换成了segment。

具体就是：

segment entry <=等价于=> inode entry

segment page <=等价于=> inode page

### 2.1、segment 和 segment entry

segment是一个逻辑上的概念，segment管理的空间也不要求连续。一个segment包括32个碎片页和三个extent的链表（FREE exent链表、NOT FULL extent链表、FULL extent链表）。segment的元数据是segment entry（实际代码中是inode entry）。

<img src="https://gongpengjun.com/imgs/innodb/innodb_segment_inode_entry.svg" width="100%" alt="xdes page">

一个segment entry占用192Byte，这些segment entry存放在哪儿呢？

### 2.2、segment page 和 segment entry 和 segment

InnoDB用专门的页来存储segment entry，这种页叫做segment page，一个segment page可以存85个segment entry（即管理85个segment）。

<img src="https://gongpengjun.com/imgs/innodb/innodb_segment_inode_page.svg" width="100%" alt="xdes page">

## 3、索引根节点页

InnoDB中数据就是索引，索引就是数据。一个聚簇索引就是一颗B+树，一个二级索引也是一颗B+树。

为了提高访问速度，每颗B+树的叶子节点页放在Leaf Node Segment中管理，内部(非叶)子节点页放在Internal Node Segment中管理。

每颗B+树的根节点页里面包含这两个segment的segment entry的位置（指针），这个信息就是FSEG Header。

<img src="https://gongpengjun.com/imgs/innodb/FSEG_Header.png" width="100%" alt="FSEG Header">

## 4、参考

- [InnoDB : Tablespace Space Management](https://dev.mysql.com/blog-archive/innodb-tablespace-space-management/)
- [Extent Descriptor Page of InnoDB](https://dev.mysql.com/blog-archive/extent-descriptor-page-of-innodb/)
- [Page management in InnoDB space files](https://blog.jcole.us/2013/01/04/page-management-in-innodb-space-files/)
- [The basics of InnoDB space file layout](https://blog.jcole.us/2013/01/03/the-basics-of-innodb-space-file-layout/)
- [Data Organization in InnoDB](https://planet.mysql.com/entry/?id=36803)
- [Data Organization in InnoDB.pdf](http://www.royalwzy.com/wp-content/uploads/2015/11/Data-Organization-in-InnoDB.pdf)
- [『译』InnoDB中的数据组织](http://pollilog.cn/2017/10/31/innodb-data-organization/)
