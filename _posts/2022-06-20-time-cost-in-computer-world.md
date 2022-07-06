---
layout: post
title: 在计算机眼中的时间
date:   2022-06-20 19:00:00
categories: Computer
---

在计算机组成的世界里，时间飞快，比飞还快。

<img src="https://gongpengjun.com/imgs/computer_time_cost.jpg" width="100%" alt="time cost in computer">

## 1、屏幕刷新 - 16.7毫秒

日常看到的电脑、平板电脑、手机的显示器的刷新频率都是60Hz，也就是一秒绘制屏幕60次，1秒/60Hz=1000毫秒/60≈16.7毫秒。

也有一些例外：

- 电影的帧率是24Hz，也就是一秒绘制屏幕24次，1秒/24Hz=1000毫秒/24≈42毫秒。
- 游戏的帧率可能会高达120Hz，也就是一秒绘制屏幕120次，1秒/120Hz=1000毫秒/120≈8毫秒。
- 最新款的手机为了让用户体验更好的游戏效果，也会提供120Hz的屏幕刷新率。

## 2、SSD刷盘速度 - 一秒4000次

| fsync rate | fsync latency | 备注           |
| ---------- | ------------- | -------------- |
| 4044/s     | 0.247ms       | Intel SATA SSD |
| 19720/s    | 0.051ms       | Intel NVMe SSD |

SATA 盘的 QPS 是 4000, NVMe 的 QPS 是 20000


## 3、网络传输 - 亚光速

### 3.1、本机网络传输 - 0.01毫秒



### 3.2、Docker网络传输 - 0.01毫秒



### 3.3、远程网络传输 - 北京到广州往返



## 4、参考资料

- 120-Hz Refresh Rate: https://www.wired.com/story/high-refresh-rate-explained/
- 120hz刷新率: https://zhuanlan.zhihu.com/p/245723447
- 企业级SSD硬盘fsync速度: https://www.ideawu.net/blog/archives/1212.html

