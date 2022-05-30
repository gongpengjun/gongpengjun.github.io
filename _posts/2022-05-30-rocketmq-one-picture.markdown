---
layout: post
title: RocketMQ一张图
date:   2022-05-30 09:00:00
categories: MQ
---

RocketMQ是阿里巴巴捐给Apache基金会的开源消息队列服务，是经历过双十一验证的消息服务中间件。

- - -

## 1、RocketMQ一览

<img src="https://gongpengjun.com/imgs/rocketmq_in_practice.svg" width="100%" alt="RocketMQ in Practice">

RocketMQ设计特点：

- 主从Broker设计，保证数据高可靠高性，拉取方式同步，只同步commitlog，然后基于commitlog异步构建消费队列Queue。
- Producer将消息均匀写入多个主Broker中，Consumer从多个主Broker中消费。
- 单台Broker中，所有Producer生产的消息存储在一个逻辑commitlog文件中，底层磁盘存储时每写满1GB数据开启一个新的文件。
- 单台Broker中，每个Topic下挂多个ConsumeQueue(简称Queue)，Producer均匀写入各个Queue，Consumer从各个Queue消费。
- 单台Broker中，一个Queue只索引一个Topic的部分消息，一个Topic的消息会均匀分布在多个Queue中。
- 一个Queue可以同时被多个Consumer Group消费，每个Consumer Group中只有一台Consumer可以消费该Queue的消息。
  - 一个Consumer Group中不能有多个Consumer消费同一个Queue的消息。
  - 一台Consumer可以消费多个Queue的消息。如Broker中有两个Queue但Consumer Group中只有一个Consumer，则该consumer会消费两个Queue的消息。
- Consumer消费的位置记录信息保存在Broker的consumerOffset.json文件中
  - consumerOffset 是一个嵌套Map：`{ "${topic}@${consumer_group}" : { ${QueueId} : ${consumerOffset} }`
  - 文件路径 `$HOME/store/config/consumerOffset.json`
  - consumerOffset.json文件内容示例：

```json
{
  "Topic A@Consumer Group 0": {
    "0": 101,
    "1": 201
  },
  "Topic B@Consumer Group 0": {
    "0": 300,
    "1": 401
  },
  "Topic B@Consumer Group 1": {
    "0": 301,
    "1": 400
  }
}
```

## 2、参考

- 你管这破玩意儿叫 MQ?  https://mp.weixin.qq.com/s/hd8z2F1hLbrykxZL6gQb1g
- RocketMQ Design: https://github.com/apache/rocketmq/blob/develop/docs/cn/design.md

