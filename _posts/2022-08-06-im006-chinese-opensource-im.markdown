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
- 服务端
  - 语言：Java
  - 存储：MongoDB、Redis、MinIO
  - 网络：Netty
  - [架构设计](https://turms-im.github.io/docs/for-developers/architecture.html)
    - 读扩散消息模型
    - 追求高性能，牺牲功能丰富性
  - 面向大用户量的C端应用场景（不是面向B端企业通讯场景）
    - 支持推模式、拉模式与推拉模式
    - 支持敏感词过滤（基于双数组Trie的AC自动机算法实现）
    - 支持消息冷热分离存储
    - [可观测性设计](https://turms-im.github.io/docs/for-developers/observability.html)
- 客户端SDK
  - turms-client-js：JS客户端SDK，支持浏览器标签页共享WebSocket连接
  - turms-client-kotlin：Android客户端SDK
  - turms-client-swift：iOS客户端SDK
  - turms-client-dart：Flutter客户端SDK
- 客户端UI
  - 不打算出带UI的客户端Demo

- 不支持离线推送

### 2、Wildfire

- [Wildfire](https://wildfirechat.cn/)：开源创业公司
- [服务端](https://github.com/wildfirechat/im-server)
  - 语言：Java
  - 存储：MySQL
  - 移动端全部免费，PC/Web端收费
  - 基于MQTT二次开发的类似于邮件的私有协议，数据使用Protobuf序列化
  - 只提供SDK，不提供完整产品
- 客户端使用微信[mars](https://github.com/tencent/mars)连接库
- 服务端
  - 一个是社区版，是开源免费单机部署
  - 一个是专业版，是闭源收费集群部署

### 3、OpenIMSDK

- [OpenIMSDK](https://doc.rentsoft.cn/#/)：开源创业公司
- [服务端](https://github.com/OpenIMSDK/Open-IM-Server)
  - 语言：Go
  - 存储：MySQL、MongoDB、Redis、MinIO
  - 网络：Netty
- 客户端
  - [Open-IM-SDK-Core](https://github.com/OpenIMSDK/Open-IM-SDK-Core)：Go语言实现的跨端SDK
  - iOS、Android、Flutter、Web、Uniapp版封装SDK
