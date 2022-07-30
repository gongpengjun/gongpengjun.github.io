---
layout: post
title: Slack线程话题模型
date:   2022-06-15 09:00:00
categories: IM
---

本文解读Slack的Thread功能，也叫话题功能的产品设计模型，基于Slack官方博客。

- - -

### 0、Thread概念

在群聊中，有这么一个场景，一部分人讨论一个话题，另一部分讨论另一个话题。两个话题的消息就会交错排在一起，比较混乱、相互打扰、难以聚焦。为了解决这个问题，就提出了Thread话题这个概念，就像一根针线把一个话题的相关消息串联起来一样(如下图所示)。

<img src="https://gongpengjun.com/imgs/im707/threads.webp" width="40%" alt="a thread of messages">

下面就解读一下Slack尝试将同一个话题串联起来的产品方法和经验教训吧。

## 探索 1 - Twitter模型，inline

- 要点：将一个thread的消息折叠，需要查看时展开。

- 教训：内测发现，每次折叠展开，都需要移动实现焦点，很破坏注意力，实际用户体验很不好。

## 探索 2 - Twitter模型，overlay

- 要点：点击消息进入thread在overlay弹出层里展示

- 教训：不断打开和关闭overlay，很多点击，容易误点

## 探索 3 - Twitter模型，sidebar

- 要点：Thread在侧边栏展示

- 反馈：**_反馈开始从关于用户界面的评论转向关于用户体验的评论，人们开始使用 Threads_**

- 教训：twitter模型里，回复一条消息形成一个thread，回复本身可以再回复又行程一个thread，设计太复杂，大量困扰，人们根本不懂怎么用Thread这个功能。

## 探索 4 – Thread单消息流

- 特点：每个thread不会分叉，只有一个消息流
- 经验：更容易理解和使用
- 教训：人们使用Thread过多，Thread的消息全部出现在主群，主群里的消息缺乏上下文，一条消息在主群和侧边栏多次展示、多次阅读，和最初Thread功能的初衷提高沟通效率和体验正好相反。

## 探索 5 – Thread侧边栏对话
- 特点：Thread消息只在侧边栏显示，不在主群显示
- 优点：**_主群隐藏Thread消息的回复被证明是设计 Thread 时所做的最有意义的改变。_**
主群和Thread的消息易于阅读和集中注意力
- 教训：Thread需要单独新增一个消息通知机制，在侧边栏右上角显示未读计数，很容易被人忽略，不符合在左上角提示的习惯。

## 探索 6 – All Threads入口

- 特点：通过 "All Threads" 作为找回Thread的入口，也同时承载了未读数的展示
- 经验：All Threads 使通知用户有新的回复变得更容易，并且用户更容易浏览最近的回复。
- 教训：暂无

**_收到的反馈主要是关于添加更多功能——这是一个积极的信号_**

## 探索 7 – 双显 Broadcasts

- 特点：允许在Thread里发消息时勾选同时发到主群里
- 经验：解决人们怕漏消息的问题
- 缺点：暂无

## 参考资料

- [Threads in Slack: A Long Design Journey (Part 1 of 2)](https://slack.design/articles/threads-in-slack-a-long-design-journey-part-1-of-2/)
- [Threads in Slack, a long design journey (part 2 of 2)](https://slack.design/articles/threads-in-slack-a-long-design-journey-part-2-of-2/)
