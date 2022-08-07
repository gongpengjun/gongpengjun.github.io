---
layout: post
title: 全球开源IM一览
date: 2022-07-30 09:00:00
categories: IM
---

IM开源实现一览，重点关注技术栈。

### 1、MatterMost

- [Mattermost](https://mattermost.com/)：对标Slack的开源实现，基于IM，瞄准协同办公场景。
- [GitHub](https://github.com/mattermost)：
  - [服务端](https://github.com/mattermost/mattermost-server)：
    - 语言：Golang
    - 架构：单体应用
    - 存储：MySQL
    - 23.6K Stars
  - 客户端：
    - [桌面端](https://github.com/mattermost/desktop)
      - 框架：Electron
      - 语言：JavaScript
      - 平台：Mac、Windows、Linux
      - 1.7K Stars
    - [移动端](https://github.com/mattermost/mattermost-mobile/tree/gekidou)
      - 框架：React Native
      - 语言：JavaScript
      - 平台：iOS、Android
      - 1.6K Stars
- 商业模式：专业版和企业版收费，搜索等功能需付费。

### 2、Signal

- [Signal](https://signal.org/)：对标Telegram的开源实现。
- [GitHub](https://github.com/signalapp)：
  - [服务端](https://github.com/signalapp/Signal-Server)：
    - 语言：Java
    - 存储：[DynamoDB](https://aws.amazon.com/cn/dynamodb/)
    - 7.8K Stars
  - 客户端：
    - [桌面端](https://github.com/signalapp/Signal-Desktop)：
      - 语言：TypeScript
      - 平台：Mac、Windows、Linux
      - 12.6K Stars
    - [iOS端](https://github.com/signalapp/Signal-iOS)
      - 语言：Swift
      - 9.4K Stars
    - [Android端](https://github.com/signalapp/Signal-Android)
      - 语言：Java
      - 22.6K Stars
  - [Mock-Signal-Server](https://github.com/signalapp/Mock-Signal-Server)
    - 语言：TypeScript
    - 用途：桌面端集成测试

### 3、Matrix

- [Matrix](https://matrix.org/): 去中心化的IM系统，类Git分布式设计
  - [Matrix.org Foundation](https://matrix.org/foundation/) 2013年发起
  - [Matrix Spec 1.0](https://matrix.org/blog/2019/06/11/introducing-matrix-1-0-and-the-matrix-org-foundation)：2019-06-11 Spec 1.0发布
- [服务端](https://github.com/matrix-org/synapse)：
  - 语言：Python 3/Twisted
  - 9.7K Stars
  - 架构：分布式、去中心化

```
client <----> homeserver <=====================> homeserver <----> client
       https://somewhere.org/_matrix      https://elsewhere.net/_matrix
```

- 客户端
  - [网页版](https://github.com/vector-im/element-web/)
    - [网页SDK](Matrix SDK for React Javascript)
    - 语言：JavaScript
    - 8.6K Stars
  - [桌面端](https://github.com/vector-im/element-desktop)：
    - 框架：Electron
    - 内核：Element 网页版
  - [iOS端](https://github.com/vector-im/element-ios)：
    - 语言：Swift + ObjectiveC 
    - 1.4K Stars
  - [Android端](https://github.com/vector-im/element-android)：
    - 语言：Kotlin
    - 2.2K Stars

### 4、Mars - [仅客户端]

- [Mars](https://github.com/Tencent/mars)：
  - 腾讯微信团队开源跨平台客户端SDK
  - 语言：C/C++
  - 平台：Android、iOS、Mac、Windows
  - 16.3K Stars
  
- [服务端Sample](https://github.com/Tencent/mars/tree/master/samples/Server)：
  - 语言：Python
  - 用于体验Mars客户端Demo


### N、参考资料

- [Google Doc - Digital Communications Protocols - IM一览表](https://docs.google.com/spreadsheets/d/1-UlA4-tslROBDS9IqHalWVztqZo7uxlCeKPQ-8uoFOU/edit#gid=0)