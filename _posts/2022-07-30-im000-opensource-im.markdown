---
layout: post
title: 开源IM一览
date: 2022-07-30 09:00:00
categories: IM
---

IM开源实现一览，重点关注技术栈。

### 1、MatterMost

- [Mattermost](https://mattermost.com/)：对标Slack的开源实现，基于IM，瞄准协同办公场景。
- [GitHub](https://github.com/mattermost)：
  - [服务端](https://github.com/mattermost/mattermost-server)：Go语言单体应用，存储MySQL, 23.6K Stars
  - 客户端：
    - [桌面端](https://github.com/mattermost/desktop)基于[Electron](http://electron.atom.io/)，JavaScript语言，支持Mac、Windows、Linux 1.7K Stars
    - [移动端](https://github.com/mattermost/mattermost-mobile/tree/gekidou)基于React Native，JavaScript语言，支持iOS和Android 1.6K Stars
- 商业模式：[专业版和企业版收费](https://mattermost.com/pricing/)，高级功能（如搜索）只在收费版提供。

### 2、Matrix

- [Matrix](https://matrix.org/): 对标Telegram的开源实现，去中心化的IM系统，技术上类似Git的分布式设计。
  - 来自剑桥大学的人创立的[Matrix.org Foundation](https://matrix.org/foundation/) 2013年发起
  - [Matrix Spec 1.0](https://matrix.org/blog/2019/06/11/introducing-matrix-1-0-and-the-matrix-org-foundation)：2019-06-11 Spec 1.0发布
- [服务端](https://github.com/matrix-org/synapse)：分布式架构，homeserver使用Python 3/Twisted开发 9.7K Stars

```
client <----> homeserver <=====================> homeserver <----> client
       https://somewhere.org/_matrix      https://elsewhere.net/_matrix
```

- 客户端
  - [网页版客户端 - Element](https://github.com/vector-im/element-web/) 8.6K Stars
    - [网页SDK](Matrix SDK for React Javascript)
  - [桌面端](https://github.com/vector-im/element-desktop)：Element Web + Electron
  - [iOS端](https://github.com/vector-im/element-ios)：Swift + ObjectiveC 1.4K Stars
  - [Android端](https://github.com/vector-im/element-android)：Kotlin 2.2K Stars

### 3、Mars - [仅客户端]

- [Mars](https://github.com/Tencent/mars)：腾讯微信团队开源跨平台客户端SDK，基于C/C++实现，16.3K Stars
  - 支持Android、iOS、Mac、Windows
- [服务端Sample](https://github.com/Tencent/mars/tree/master/samples/Server)：Python实现的服务端样例，用于体验Mars客户端Demo

### N、参考资料

- [Google Doc - Digital Communications Protocols - IM一览表](https://docs.google.com/spreadsheets/d/1-UlA4-tslROBDS9IqHalWVztqZo7uxlCeKPQ-8uoFOU/edit#gid=0)