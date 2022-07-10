---
layout: post
title: Netty内存池无锁设计
date:   2022-07-08 09:00:00
categories: netty
---

本文是博客文章《详解 Recycler 对象池的精妙设计与实现》的摘要，只关注Netty无锁化设计部分。

- - -

## 1、常规锁同步设计

<img src="https://gongpengjun.com/imgs/netty/netty_1.webp" width="100%" alt="extent and xdes entry">

锁同步发生场景：

- 多个线程thread1, thread2同时从对象池获取对象时，需要锁来同步管理对对象池的写操作，否则可能出现两个线程同时获取到对象池中同一个对象的问题。
- 一个线程thread1从对象池获取4个对象后，将其分别交给另4个线程thread2、thread3、thread4、thread5使用，那么当thread2、thread3、thread4、thread5用完释放对象时，需要锁来同步管理对对象池的写操作，否则可能存在对象池数组中元素相互覆盖的问题。

## 2、Netty无锁化设计

### 2.1、数据结构

数据结构是代码分析的起点。

每个Thread一个Stack对象，每个Thread获取对象和释放对象都操作本thread对应的Stack：

<img src="https://gongpengjun.com/imgs/netty/netty_2.webp" width="100%" alt="extent and xdes entry">



每个Thread的Stack持有一个WeakOrderQueue组成的链表，链表中的每个WeakOrderQueue节点仅供一个Thread使用，如上图所示：第一个WeakOrderQueue节点仅供Thread2使用，第二个WeakOrderQueue节点仅供Thread3使用，第三个WeakOrderQueue节点仅供Thread4使用，以此类推。这样Thread2、Thread3、Thread4之间就不会相互竞争了。



一个WeakOrderQueue内部的结构如下：（注：下图的head和上图的head无关）

<img src="https://gongpengjun.com/imgs/netty/netty_3.webp" width="100%" alt="extent and xdes entry">



有了数据结构的布局图，分析起来就得心应手了。

## 3、无锁设计要点分析

我们以Thread1获取对象obj1交给Thread2使用，然后Thread2用完objc后将其释放的过程来分析无锁化是怎么达成的。

### 3.1、Thread1获取池化对象obj1

Thread1从对象池获取一个obj1，对象池返回的对象是obj1的包装对象Handle。Handle中会记录创建线程thread1的信息。

### 3.2、Thread1将池化对象obj1交给Thread2

常规的线程间交互，无特殊处理。

### 3.3、Thread2归还池化对象obj1 - pushLater

Thread2用户对象obj1后，准备归还。

- 首先Thread2根据obj1的包装对象Handle中的记录的信息找到其创建线程thread1，然后遍历thread1的stack1管理的WeakOrderQueue链表，找到专属于Thread2的WeakOrderQueue（如果没有就创建一个并插入该链表）。

- 然后Thread2通过WeakOrderQueue的tail指针找到最后一个Link对象，再往Link对象的elements数组最后追加一条记录。
  - WeakOrderQueue的tail指针总是指向最后一个Link对象
  - 往Link对象的elements数组最后追加一条记录实际过程：
    - 先获取该Link的size，即为writeIndex
    - 然后往数组中该位置插入一条记录
    - 最后将该Link的size加一

### 3.4、Thread1释放池化对象obj1 - scavenge→transfer

Thread1访问自己Stack中的WeakOrderQueue链表，一次一个WeakOrderQueue节点的速度来释放其中的对象。

- 首先Thread1通过其对应Stack中的head指针找到第一个Link节点
- 然后从该Link的readIndex位置开始将Link的elements中的对象转全部移回Stack的elements数组中
- 最后分两种情况
  - 如果该Link中的16个elements全都处理完毕，又分为两种情况
    - 如果当前Link不是最后一个Link `Link.next != null` 则将当前Link从链表移除
    - 如果当前Link是最后一个Link  `Link.next != null` ，则暂不从将当前Link从链表移除（避免和Thread2同时修改同一个Link的`Link.next`指针）
  - 如果该Link中的16个elements未处理完毕，则该Link的readIndex位置更新到最后一个elements处
    - 比如，当前Link中只有3个对象，则将这3个对象一次性回收完，将readIndex从0移到3的位置

**总结**：经过精心的安排，将可能的资源竞争消除掉

- 获取对象时，分两种情况
  - 如果自己的Stack有对象，则各个线程从自己的Stack获取，不会相互竞争
  - 如果自己的Stack无对象，则各个线程从自己的Stack的WeakOrderQueue的**头Link节点**中回收一批对象
    - Thread1只访问**头Link节点**，只读取`head.link.readIndex`回收对象并修改`head.link.readIndex`
    - Thread1只在Link节点组成的链表有多个Link节点时才修改`head.link.next`，保证不会和Thread2修改同一个Link节点的`Link.next`指针
- 释放对象时，分两步走
  - 线程Thread2先将对象放回Thread2专属的WeakOrderQueue的tail指向的Link节点的elements数组最后（无需访问readIndex）
    - 如果tail.elements满16个对象了，则新创建一个Link节点并把对象放进去，将该Link挂在链表最后且更新tail指向该Link节点
      - 次操作会读取：`tail`和`tail.writeIndex`
      - 此操作会修改：`tail`和`tail.next`和`tail.writeIndex`


## 4、参考资料

- [详解 Recycler 对象池的精妙设计与实现](https://xie.infoq.cn/article/ea5c220d79a131a2fbe57f142)
