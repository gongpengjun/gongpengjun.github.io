---
layout: post
title: 网络抖动、丢包、延迟、拥塞
date: 2023-03-26 09:00:00
categories: network
---

网络抖动、丢包、延迟、拥塞是什么?

### 1、网络抖动

网络抖动好比突发性的堵车，主要是线路负载过大造成的。

<img src="https://gongpengjun.com/imgs/network/network_jitter.gif" width="100%" alt="network jitter">

### 2、网络丢包

当线路上的数据太多，运营商网络处理不过来的时候，一些数据就会丢失。

<img src="https://gongpengjun.com/imgs/network/packet_drop.gif" width="100%" alt="packet drop">

- 自动重传请求（ARQ）

重传：自动重传请求（Automatic Repeat-reQuest, ARQ）是错误纠正协议，主控端在发现丢包时，会告知被控端哪些数据丢失，请求重传。被控端收到请求后再次发送丢包的数据，最终补全原有信息。

<img src="https://gongpengjun.com/imgs/network/packet_drop_arq.gif" width="100%" alt="ARQ">

- 前向纠错算法（FEC）

冗余： 前向纠错（Forward Error Correction, FEC）算法，就像飞鸽传书时一次放好几只，即使有一两只飞丢，其余信鸽也能把信息送达。

<img src="https://gongpengjun.com/imgs/network/packet_drop_fec.gif" width="100%" alt="FEC">

由发送端对原始数据进行 FEC 编码，生成冗余奇偶校验数据包，原始数据和冗余数据包合并称作 FEC 数据块，发送端发送 FEC 数据块。接收端接收到 FEC 数据块后，通过冗余数据包和原始数据包来恢复出丢失或者出错的数据包。


### 3、网络延迟

数据包转发就像快递在集散中心分拣要花时间一样，数据包在机房转发时也有处理时间。如果处理不及时，还会增加数据包的排队时间。

<img src="https://gongpengjun.com/imgs/network/packet_delay.gif" width="100%" alt="packet delay">

### 4、网络拥塞

网络线路都有承载上限，如果线路上的数据超载，传输性能就会急剧下降，出现拥塞和弱网。就像是一条三车道公路，并排开一至三辆车都不拥堵，而一旦塞入第四辆，所有车速都会急剧下降，陷入拥堵。

<img src="https://gongpengjun.com/imgs/network/congestion_control_no.gif" width="100%" alt="no congestion control">

采用拥塞控制算法来合理利用带宽:

- 基于延时的算法（Delay-based）：通过RTT采样和Kalman-Filter来监测延时，根据延时变动来预测网络负载变化
- 基于丢包的算法（Loss-based）：对随机丢包、拥塞丢包和突发丢包进行智能识别，避免线路因随机丢包被错判为拥塞

<img src="https://gongpengjun.com/imgs/network/congestion_control_yes.gif" width="100%" alt="have congestion control">



### 参考

- https://www.todesk.com/news/411.html
- https://www.todesk.com/news/410.html
- https://www.todesk.com/news/413.html
- https://www.todesk.com/news/420.html
- http://tflach.github.io/net-tcp-fec/
- https://www.rfc-editor.org/rfc/rfc5109
