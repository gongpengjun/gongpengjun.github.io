---
layout: post
title: IM007 - 兼容性设计
date: 2022-08-17 09:00:00
categories: IM707
---

IM是一个持续迭代的系统，很容易同时并存多个版本，而服务端只有一个版本，怎么做好新老版本兼容是一个很重要的话题。

## IM 客户端版本分布和IM系统拓扑图

IM是端到端系统，也是以IM Server为中心的星型系统，一个IM系统的客户端不同版本的分布和服务端形成的拓扑图如下：

<img src="https://gongpengjun.com/imgs/im707/im007_client_version_distribution.svg" width="60%" alt="IM007-IM Client Version Distribution">

- V1: 较老的客户端版本有一定量的人在用，因为懒不想升级或者客观原因无法升级；

- V2: 最近发布的版本有比较多的人在用，是主力版本；
- V3: 正在研发的版本是Next Big Thing，拥有“Killer Feature”，有很多Breakthrough，也有很多breaking chanages
  - 下一个版本要搞个大事情
  - 拥有杀手级功能
  - 有很多突破
  - 自然也有很多不兼容性改变

不能让用户从V1强升到V2或者V3，对于V2也只能引导其升到V3，而且V3开发耗时数月心里没底儿不敢一下子让用户大规模升上来，怎么办？只能兼容老版本，V1还有V2。



要兼容，就先需要判断新老版本，怎么判断新老版本就是关键。

## 版本标识

**选什么做版本标识呢？**

- 最直接的选择是App Version，比如微信Version 8.0.25这种常见的版本号，字符串格式。
- 还有一个选择是App Version Code，一个整数的版本号，方便比对判断，而且App Version Code不是用户可见的，可以对相同Version不同build赋不同的Version Code，比如有小的Bug Fix可以保持Version不变只改动Version Code。在一套IM多套部署和多个App的场合下，App Version Code不能跨App使用是个弊端。
- 支持跨App使用的办法就是在App Version、App Version Code之外加个IM Version Code，不同App使用同一套IM Version Code，这样可以跨App使用，不同App可以使用同一套判断逻辑来对某个功能做的新老版本做不同的处理。

**换个视角看选择**：

- 站在客户端的角度看版本标识选择，客户端不想同时维护多个版本号：App Version、App Version Code、IM Version Code，多个版本号很容易一个升级另一个没升级，造成漏配、错配，维护成本高。客户端希望只维护一个App Version或者App Version Code（iOS和Android系统都要求有这两个版本号）。
- 站在服务端的角度看版本标识选择，服务端想让客户端维护一个跨App、跨多端、且类型是整数的IM Version Code，这样服务端解析方便、使用方便、判断准确。

**再换个角度看选择**：

- 做兼容往往是针对某个Feature来做的，能不能针对每个Feature设计Feature Version Code呢？当然是能的，但是不太好，因为Feature会越来越多，也就需要越来越多的版本标识。而多个Feature Version Code**等价于**一个统一的整个IM SDK或者IM App的统一的Version Code。

**再换个角度看选择**：

- 站在产品经理角度：我想使用直观能看到的App Version，这样沟通方便，再搞个其它的版本标识我还要一一对应，麻烦。
- 站在客户端角度：我想使用能直观看到的App Version或App Version Code，我只要维护一个版本标识就好，不容易出错。
- 站在服务端角度：我想使用多App一致、多端一致、解析方便、准确，容易比对的整数版本如IM SDK Version Code。

**最后的抉择**：

其实版本标识选择更多要考虑人的因素、可持续性维护、多端统一性。

- 如果有个跨端的IM SDK，那么统一的IM SDK Version Code可以让所有人：产品经理、IM客户端SDK研发、IM服务端研发都满意。

- 如果是各端独自实现IM，那么复用统一的App Version或者App Version Code能让大多数人（产品经理和多个客户端研发）满意。

## 版本通道

#### 短连接：

通过HTTP Header **UserAgent**增加统一的字段传递，示例如下:

```shell
curl "https://{domain}/api/v1/xxx" \
	-H "UserAgent: ... platform/ios, version/8.0.25 ..." \
	-H "traceid: 0ad1348f1403169275002100356696"
```

一般来说，客户端的HTTP网络库都会开放UserAgent的修改接口，所以复用UserAgent比较容易实现。

#### 长连接：

通过类似HTTP Header的长链接的数据包中的header来传和UserAgent相同的字段和值，示例如下：

```json
{
  "platform": "ios",
  "version": "8.0.25",
  "traceid": "0ad1348f1403169275002100356696"
}
```

#### 短转长：

IM系统有长连接通道，为了优化HTTP，往往会通过**短转长**方案，来节省TCP建连和TLS握手时间来提高访问速度和请求成功率。

在短转长场景下，短连接已经传了一遍platform和version，长连接SDK也会再传一遍，从业务角度，应该用短链传的，这样业务服务就不用区分请求来自短链还是长链。



## 求指点

关于IM或者长期维护的App的兼容性建设，你有什么好的建议或者方案呢？求指点。可以加我微信gongpengjun或者给我发邮件frank.gongpengjun at gmail.com

