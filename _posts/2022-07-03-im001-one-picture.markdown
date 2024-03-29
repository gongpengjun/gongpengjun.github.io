---
layout: post
title: IM001 - 三纵三横
date: 2022-07-02 09:00:00
categories: IM707
---

IM系统是一个复杂的端到端系统，其业务本质要求系统必须是三纵三横的架构。

<img src="https://gongpengjun.com/imgs/im707/im001.png" width="100%" alt="IM001-三纵三横">

IM三横：

- 【上】用户层：客户端IM UI层负责消息展示和用户交互，发消息、收消息、未读数提醒等。
- 【中】数据层：客户端IM SDK负责消息的本地持久化和消息重发及消息去重，服务端IM消息服务负责消息的持久存储和有序性。
- 【下】网络层：客户端长连接SDK跟服务端长连接网关服务保持长连接状态，保证IM消息的实时性。

IM三纵：

- 【左】发送端：发IM消息由发送客户端端负责，通常有iOS、Android、Windows、Mac、网页版、微信小程序等等平台的实现。
- 【中】服务端：IM消息收发都由服务端中转，服务端负责消息的接收、推送（在线推送、离线推送）、拉取（断线重连）。
- 【右】接收端：收IM消息由接收客户端负责，通常有iOS、Android、Windows、Mac、网页版、微信小程序等等平台的实现。
