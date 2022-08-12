---
layout: post
title: REST API的坑
date: 2022-08-12 09:00:00
categories: API
---

REST API普遍被推崇为更好的HTTP API风格，但其也有不利的方面。

### 基础设施适配难

大多数公司里的基础架构有关于服务可观测性的平台，比如

- 微服务trace日志
- 链路聚合分析
- 接口错误聚类
- 服务拓扑关系
- 接口实时监控

都需要一个简单唯一的uri来聚合同一类api，而REST API的path里带有参数，往往很难利用现有的基础平台的能力。

### 交流沟通麻烦

试想一个场景：测试同学发现一个Bug，反馈说：“/api/v1/posts接口返回400了”，如果是REST API，那么需要询问是请求方法是GET、POST、PUT、DELETE中的哪个？测试、前端、后端等多角色参与时，沟通成本明显增加。

反观传统的API风格，测试同学发现一个Bug，反馈说：“/api/v1/post.create接口返回400了”，那么大家立刻就理解这是说的哪个场景，沟通效率就提升了。

### 参考资料

[什么是 RESTful API？](https://aws.amazon.com/cn/what-is/restful-api/)
