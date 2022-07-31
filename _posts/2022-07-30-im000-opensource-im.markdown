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

### 2、Signal

- [Signal](https://signal.org/)：对标Telegram的开源实现。
- [GitHub](https://github.com/signalapp)：
  - [服务端](https://github.com/signalapp/Signal-Server)：Java语言，存储[DynamoDB](https://aws.amazon.com/cn/dynamodb/)， 7.8K Stars
    - [Mock-Signal-Server](https://github.com/signalapp/Mock-Signal-Server)：TypeScript语言，仅用于桌面端集成测试 7 Stars
  - 客户端：
    - [桌面端](https://github.com/signalapp/Signal-Desktop)：TypeScript语言，支持Mac、Windows、Linux 12.6K Stars
    - [iOS端](https://github.com/signalapp/Signal-iOS)，Swift语言 9.4K Stars
    - [Android端](https://github.com/signalapp/Signal-Android)，Java语言 22.6K Stars

### 3、Matrix

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



### 4、Mars - [仅客户端]

- [Mars](https://github.com/Tencent/mars)：腾讯微信团队开源跨平台客户端SDK，基于C/C++实现，16.3K Stars
  - 支持Android、iOS、Mac、Windows
- [服务端Sample](https://github.com/Tencent/mars/tree/master/samples/Server)：Python实现的服务端样例，用于体验Mars客户端Demo

### 4、Turms

- [Turms](https://github.com/turms-im/turms)：个人开发者，Java语言，号称支持10万到千万并发用户
  - 存储：MongoDB、Redis
  - Turms基于读扩散消息模型进行架构设计，源自商用即时通讯项目
  - 支持推模式、拉模式与推拉模式
  - [架构设计](https://turms-im.github.io/docs/for-developers/architecture.html)
  - [可观测性设计](https://turms-im.github.io/docs/for-developers/observability.html)

### 5、Wildfire

- [Wildfire](https://github.com/wildfirechat)：创业公司，开源IM，基于Java语言，
  - 存储：MySQL
  - [支持单机百万链接](https://github.com/wildfirechat/C1000K_Test)
  - [野火IM的商业逻辑](https://docs.wildfirechat.cn/blogs/%E9%87%8E%E7%81%ABIM%E7%9A%84%E5%95%86%E4%B8%9A%E9%80%BB%E8%BE%91.html)
  - 移动端全部免费
  - 对PC/Web端收费
  - 基于MQTT二次开发的类似于邮件的私有协议，数据使用Protobuf序列化
  - 团队核心成员从事IM行业10年以上
  - 只提供SDK，不提供完整产品
- 客户端使用微信[mars](https://github.com/tencent/mars)连接库
- 服务端
  - 一个是社区版，是开源免费单机部署
  - 一个是专业版，是闭源收费集群部署

### 6、OpenIMSDK

- [OpenIMSDK](https://github.com/OpenIMSDK)：创业公司，Go语言实现
- OpenIM是由IM技术专家打造的**开源**的即时通讯组件。OpenIM包括IM服务端和客户端SDK
- 存储：MySQL、MongoDB、Redis

### N、参考资料

- [Google Doc - Digital Communications Protocols - IM一览表](https://docs.google.com/spreadsheets/d/1-UlA4-tslROBDS9IqHalWVztqZo7uxlCeKPQ-8uoFOU/edit#gid=0)
- [IM开源即时通讯软件收集](https://blog.csdn.net/libaineu2004/article/details/44026069)