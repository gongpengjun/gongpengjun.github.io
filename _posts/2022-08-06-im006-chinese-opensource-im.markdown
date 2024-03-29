---
layout: post
title: 国产开源IM
date: 2022-08-06 09:00:00
categories: IM
---

国产IM一览，重点关注开源情况和技术栈。

### 1、Mars

- [Mars](https://github.com/Tencent/mars)：
  - 腾讯微信团队开源跨平台客户端SDK
  - 语言：C/C++
  - 平台：Android、iOS、Mac、Windows
  - 16.3K Stars
  
- [服务端Sample](https://github.com/Tencent/mars/tree/master/samples/Server)：
  - 语言：Python
  - 用于体验Mars客户端Demo


### 2、Turms

- [Turms](https://github.com/turms-im/turms)：个人开发者，号称支持十万到千万并发用户
- [服务端](https://github.com/turms-im/turms)
  - 语言：Java
  - 存储：MongoDB、Redis、MinIO
  - 网络：Netty
  - [架构设计](https://turms-im.github.io/docs/for-developers/architecture.html)
    - 读扩散消息模型
    - 追求高性能，牺牲功能丰富性
    - 面向大用户量的C端应用场景（非B端企业协作）
    - 支持推模式、拉模式与推拉模式
    - 支持敏感词过滤（双数组Trie的AC自动机算法）
    - 支持消息冷热分离存储
    - [可观测性设计](https://turms-im.github.io/docs/for-developers/observability.html)
- 客户端
  - [turms-client-js](https://github.com/turms-im/turms/tree/develop/turms-client-js)：JS客户端SDK，多标签页共享WebSocket连接
  - [turms-client-kotlin](https://github.com/turms-im/turms/tree/develop/turms-client-kotlin)：Android客户端SDK
  - [turms-client-swift](https://github.com/turms-im/turms/tree/develop/turms-client-swift)：iOS客户端SDK
  - [turms-client-dart](https://github.com/turms-im/turms/tree/develop/turms-client-dart)：Flutter客户端SDK
- 客户端UI
  - 不打算出带UI的客户端Demo

- 不支持离线推送


### 3、OpenIMSDK

- [OpenIMSDK](https://www.rentsoft.cn/)：IM创业公司
  - [团队专栏](https://cloud.tencent.com/developer/column/92952)
- [服务端](https://github.com/OpenIMSDK/Open-IM-Server)
  - 语言：Go
  - 存储：MySQL、MongoDB、Redis、MinIO
    - Redis 用于消息序列号生成
    - MongoDB缓存最近消息，直接拉取消息
    - MySQL存储全量历史消息
    - MinIO存储文件
  - 网络：Netty
  - [架构设计](https://cloud.tencent.com/developer/article/1853605)：
    - 写扩散模型
- 客户端
  - [Open-IM-SDK-Core](https://github.com/OpenIMSDK/Open-IM-SDK-Core)：Go语言实现的跨端SDK
  - iOS、Android、Flutter、Web、Uniapp封装

### 4、Wildfire

- [Wildfire](https://wildfirechat.cn/)：IM创业公司
- [GitHub](https://github.com/wildfirechat)
  - [服务端](https://github.com/wildfirechat/im-server)
    - 语言：Java
    - 存储：MySQL
    - 社区版，开源免费单机部署
    - 专业版，闭源收费集群部署
  - 移动端
    - 客户端使用微信[mars](https://github.com/tencent/mars)连接库
    - [iOS端](https://github.com/wildfirechat/ios-chat)
    - [Android端](https://github.com/wildfirechat/android-chat)
    - [桌面端](https://github.com/wildfirechat/vue-pc-chat)
