---

layout: post
title: tcpdump抓包http流量
date: 2022-11-27 09:00:00
categories: 网络
---

### tcpdum http 常用命令

#### 命令一：

```shell
$ tcpdump -A -s 0 '[src]/[dst] host {HOST} and port {PORT} and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) !=0)'
```

抓取和baidu.com的80端口之间的HTTP包:
```shell
$ tcpdump -A -s 0 'host baidu.com and tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'
```

抓取8080端口的HTTP包:
```shell
$ tcpdump -A -s 0 'tcp port 8080 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) !=0)'
```

### ASCII Code

```shell
gongpengjun@mbp cheatsheat-tcpdump$ echo -n "GET " | hexdump -C
00000000  47 45 54 20                                       |GET |
00000004
gongpengjun@mbp cheatsheat-tcpdump$ echo -n "POST" | hexdump -C
00000000  50 4f 53 54                                       |POST|
00000004
gongpengjun@mbp cheatsheat-tcpdump$ echo -n "PUT" | hexdump -C
00000000  50 55 54                                          |PUT|
00000003
gongpengjun@mbp cheatsheat-tcpdump$ echo -n "HTTP" | hexdump -C
00000000  48 54 54 50                                       |HTTP|
00000004
gongpengjun@mbp cheatsheat-tcpdump$ echo -n "HEAD" | hexdump -C
00000000  48 45 41 44                                       |HEAD|
00000004
```

### 【命令解析】

### 参考资料

- https://hackertarget.com/tcpdump-examples/
- https://danielmiessler.com/study/tcpdump/
- https://danielmiessler.com/study/tcpflags/
- https://wiki.wireshark.org/CaptureFilters
- https://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure
- https://security.stackexchange.com/questions/121011/wireshark-tcp-filter-tcptcp121-0xf0-24
- https://stackoverflow.com/a/74595565/328435
