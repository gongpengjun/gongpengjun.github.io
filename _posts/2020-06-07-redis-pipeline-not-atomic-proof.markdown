---
layout: post
title: Redis Pipeline非原子性实证
date:   2020-06-07 18:00:00
categories: Redis
---

众所周知，Redis流水线[pipeline](https://redis.io/topics/pipelining)功能可以实现命令的**打包发送**。
因为Redis是单线程工作模型，那么pipeline里面的命令是否是原子执行的呢？今天通过一段测试代码和Wireshark抓包来看一下。

- - -

## 测试代码

[`test_pipeline_not_atomic.py`]()

```python
import redis
import threading

def get_func():
    r1 = redis.Redis('127.0.0.1')
    pipe = r1.pipeline(transaction=False)
    for j in range(1000):
        pipe.get('k')
    res = pipe.execute()
    for j in range(1000 - 1):
        if res[j] != res[j + 1]:
            print('not atomic')

def incr_func():
    r1 = redis.Redis('127.0.0.1')
    pipe = r1.pipeline(transaction=False)
    for j in range(1000):
        pipe.incr('k')
    pipe.execute()

if __name__ == "__main__":
    r = redis.Redis('127.0.0.1')
    r.set('k', 0)
    threading.Thread(target=get_func).start()
    threading.Thread(target=incr_func).start()
```

## 测试抓包

### 启动redis server

```
$ docker run --detach=true -p 6379:6379 redis:3.2.8 redis-server
```

### 启动抓包命令

启动Wireshark的命令行程序tshark抓取`Loopback: lo0`且TCP端口号为6379的网络包:

```
$ tshark -i lo0 -f "tcp port 6379" -w redis_pipeline.pcapng
Capturing on 'Loopback: lo0'
56 ^C
```

### 执行测试代码

```
$ python3 test_pipeline_not_atomic.py
not atomic
```

从程序的输出`not atomic`可以看到pipeline是非原子性的。

## 分析网络包

### 下载RESP协议解码器

[redis-wireshark](https://github.com/jzwinck/redis-wireshark) is Redis protocol dissector for Wireshark

```
$ wget https://raw.githubusercontent.com/jzwinck/redis-wireshark/master/redis-wireshark.lua -O ~/.local/lib/wireshark/plugins/redis-wireshark.lua
```

或

```
$ curl https://raw.githubusercontent.com/jzwinck/redis-wireshark/master/redis-wireshark.lua --output ~/.local/lib/wireshark/plugins/redis-wireshark.lua
```

如果遇到问题，也可以使用浏览器下载

### 使用REST解码器分析

```
$ tshark -r redis_pipeline.pcapng -z "conv,tcp"
```

```
$ tshark -r redis_pipeline.pcapng -z "follow,tcp,ascii,0"
===================================================================
Follow: tcp,ascii
Filter: tcp.stream eq 0
Node 0: 127.0.0.1:50522
Node 1: 127.0.0.1:6379
27
*3
$3
SET
$1
k
$1
0

	5
+OK

===================================================================
```

```
$ tshark -r redis_pipeline.pcapng -z "follow,tcp,ascii,1"
===================================================================
Follow: tcp,ascii
Filter: tcp.stream eq 1
Node 0: 127.0.0.1:50524
Node 1: 127.0.0.1:6379
6030
*3
$6
INCRBY
$1
k
$1
1
*3
$6
INCRBY
$1
k
$1
1
*3
$6
INCRBY
$1
k
$1
1
...
:1
:2
:3
:4
:5
...
:202
:203
:204
:205
:206
:207
...
:995
:996
:997
:998
:999
:1000

===================================================================
```


```
$ tshark -r redis_pipeline.pcapng -z "follow,tcp,ascii,2"
===================================================================
Follow: tcp,ascii
Filter: tcp.stream eq 2
Node 0: 127.0.0.1:50523
Node 1: 127.0.0.1:6379
6020
*2
$3
GET
$1
k
*2
$3
GET
$1
k
*2
$3
GET
$1
k
...
$3
948
$3
948
$3
948
...
$4
1000
$4
1000
$4
1000

===================================================================
```

- - -

## 结论

1. Redis Pipeline不保证原子性。
1. Redis Pipeline请求和响应是流式的。
1. Redis序列化协议(RESP)是流式协议，request/response没有开始和终止分界符。

## 进一步

使用[redis-wireshark.lua](https://github.com/jzwinck/redis-wireshark)查看抓包详情:

```
$ tshark -X ~/.local/lib/wireshark/plugins/redis-wireshark.lua -r redis_pipeline.pcapng -z "follow,tcp,ascii,0" redis
```
