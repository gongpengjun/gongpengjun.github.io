---
layout: post
title: REST API的坑
date: 2022-08-12 09:00:00
categories: API
---

REST API被推崇为更好的HTTP API风格，但其也有不好的方面。

### 基础设施适配难

公司基础架构有关于服务可观测性的平台，常见功能如下：

- 微服务trace日志
- 链路聚合分析
- 接口错误聚类
- 服务拓扑关系
- 接口实时监控

这些功能都需要唯一的uri来聚合同一个api，而REST API的path里带有参数，往往很难适配或者完整利用以上平台能力。

### 交流沟通麻烦

试想服务开发联调、上线运维过程中的一个常见场景：某人发现某个接口400了。

- 如果是REST API，当反馈的人说：“/api/posts接口返回400了”，还需要询问请求方法是GET、POST、PUT、DELETE中的哪个？测试、前端、后端、相关服务负责方等多角色参与时，多一次确认就多不少沟通成本。

- 如果是传统风格API，当反馈的人说：“/api/post.create接口返回400了”，那么立刻就能确定这是说的创建POST的场景，准确无歧义。

### 参考资料

[什么是 RESTful API？](https://aws.amazon.com/cn/what-is/restful-api/)
