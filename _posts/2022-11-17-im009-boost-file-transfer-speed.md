---

layout: post
title: IM009 - 文件传输链路择优
date: 2022-11-17 09:00:00
categories: IM
---

一天，有个IM用户发来一张文件下载速度为 0.0 B/s 的截图，并反馈：“在办公网，下载文件咋会这么慢...”

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/speed_zero.png" alt="speed zero">
</p>

这种速度让人抓狂，为什么有这么慢甚至卡住的下载速度？问题到底出在哪里？怎么解决？如果你对这些问题感兴趣，欢迎一起来看看优化过程吧。



### 【根因分析】

我们测试了移动办公、职场办公、在家办公三种场景，发现了在职场办公时文件下载特别慢，经常只有几百KB的速度。

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/speed_slow.png" alt="speed zero">
</p>

我们梳理出IM文件传输的网络拓扑结构，如下图所示：

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/network_topology_question.png"  alt="network topology">
</p>

从上图对比各种办公场景的网络链路都是走互联网到达线上机房，那么问题很可能出在差异化的链路部分，也就是（上图红色连线所示）职场办公网到互联网之间的链路可能存在特殊性，于是找IT同事沟通，得知办公网出口网络带宽小，单用户1MB/s限流，限流会导致丢包重传，会严重影响传输性能，所以文件传输很慢，甚至会卡住。

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/network_topology_answer.png" alt="network topology with bandwith">
</p>

> 可能有人会问办公网出口有限流，传输慢可以理解，为啥会卡住不动呢？
>
> 原因是在触发限流后会导致丢包重传。我们通过tcpdump抓包分析，发现问题发生时发生了TCP丢包，继而触发重传，对传输性能影响很大，会导致性能急剧下降，而IM下载速度是计算最近一秒内接收到的数据量。所以当发生丢包请求服务端重传时，会有较长时间没有新的数据到达，所以会显示速度为0.0 B/s。

### **【解决方案】**

既然问题根因是职场公网出口带宽限流导致的，那么放大限流值不就可以了么，经过跟IT同事沟通，发现职场公网出口总的带宽就比较小，职场同时办公人数多且使用集中，没有办法简单把限流值放大。

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/office_network_file_route_before.png" alt="office network route old">
</p>

扩大办公网出口带宽这条路不通，那能不能换一条路呢？直接走职场办公网到线上机房，可以吗？经过IT同事确认，国内大多数职场都会直接或者间接和线上机房建立由专线连接，带宽也比较大。具体方案如下图右侧绿色箭头所示：

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/office_network_file_route_after.png" alt="office network route new">
</p>

经过测速验证，走专线速度可大幅提高，初步确认这个方案是可行的。

### 【落地实施】

有了解决方案，落地实施就顺理成章了，但是过程中有很多细节要妥善处理，这里介绍三个要点： 

**1、选路标准**：因为只有办公网有限流和总带宽小的问题，也只有办公网有到线上机房的专线，所以需要判断IM目前处于办公网与否，发现用户处于办公网才走专线。跟IT沟通发现办公网的IP段基本是稳定的，所以我们选择通过IP地址来判断，如果IM客户端的IP地址在办公网IP池中则可以判定其处于办公网。



 **2、监控指标**：有句老话，If you can't measure it, you can't improve It. 要优化文件传输速度，我们必须首先准确地测量文件传输速度。于是，我们完善埋点，增加IP和职场维度，聚合分析出文件下载上传的速度监控指标。



 **3、灰度放量**：优化也是代码变更，凡是变更都伴随着引入Bug的风险。为了控制优化效果，我们采用灰度开关来逐步灰度放量。先在某个职场选指定的一些用户加入优化白名单，效果符合预期后，再按比例将该职场的其它用户加入优化名单，一个职场一个职场扩大灰度范围。



### 【优化效果】

优化前后使用感受：   

- 办公网链路优化前，文件上传下载速度慢，经常失败   
- 办公网链路优化后，文件上传下载速度快，成功率高

优化前办公网文件传输的速度一般是这样的：

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/speed_slow.png" alt="speed slow">
</p>

现在的办公网文件传输的速度一般是这样的：

<p style="text-align: center;">
<img src="https://gongpengjun.com/imgs/im707/file_perf/speed_fast.png" alt="speed fast">
</p>

经常还能看到10MB/s以上的传输速度，感觉可以说是嗖嗖嗖地。



优化前后监控指标显示：文件上传下载速度95分位值 由几百KB/s提高到3~5MB/s，提速了 6~7 倍。

### 【思考启示】

本文记录了一个文件卡顿问题的优化过程，虽然简单，但它给我了一些启发：

- 一个好用的工具是需要持续优化的，在改进和反馈的循环中不断变好。
- 只有通用的优化（比如分片和断点续传）可能是不够的，还要结合具体使用场景（比如职场办公外网）的特点来适配和优化（比如选择更优的网络链路）。
- 影响实际体验的才是更重要的，优先收集用户使用痛点，排列优先级处理。
