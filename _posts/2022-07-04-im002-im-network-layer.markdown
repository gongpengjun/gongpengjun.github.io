---
layout: post
title: IM002 - 网络层
date: 2022-07-03 09:00:00
categories: IM707
---

IM系统要在端到端之间实时传递消息，所以网络层是其最核心的组成部分。

<img src="https://gongpengjun.com/imgs/im707/im002.png" width="100%" alt="IM002-网络层">

短连接：

- iOS、Android、Windows、Mac、Web各用系统短连接
- HTTP(S)短连接，由客户端发起TCP连接，只能用于客户端端主动联系服务端，不能用于服务端主动联系客户端。

长连接：

- iOS、Android、Windows、Mac、Web共用一套长连接库
- TCP长连接，由客户端发起TCP连接，心跳保活和断线重连，既可以用于客户端主动联系服务端，也可以用于服务端主动联系客户端。

短转长：

- 有了长连接机制，将短连接在网络SDK层和服务端的长连接网关处做一个转换，可以在业务层还无感知的情况下，让HTTP(S)走长连接通道，节省掉前后端交互的TCP建连耗时，同时提高API接口成功率。
