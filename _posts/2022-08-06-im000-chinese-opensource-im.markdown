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

### 2、Wildfire

- [Wildfire](https://github.com/wildfirechat)：创业公司，开源IM
  - 语言：Java
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

### 3、OpenIMSDK

- [OpenIMSDK](https://github.com/OpenIMSDK)：创业公司，Go语言实现
- OpenIM是由IM技术专家打造的**开源**的即时通讯组件。OpenIM包括IM服务端和客户端SDK
- 存储：MySQL、MongoDB、Redis
