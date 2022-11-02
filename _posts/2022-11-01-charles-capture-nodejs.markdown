---
layout: post
title: Charles抓包NodeJS应用
date: 2022-11-01 09:00:00
categories: PacketCapture
---

使用Charles抓包较老版本的NodeJs应用时遇到错误码SELF_SIGNED_CERT_IN_CHAIN。

### 问题现象

Charles抓包界面报错：

```
Failure Client closed the connection before a request was made. Possibly the SSL certificate was rejected.
Notes You may need to configure your browser or application to trust the Charles Root Certificate. See SSL Proxying in the Help menu.
```

<img src="https://gongpengjun.com/imgs/network/charles_ssl_cert_rejected.png" width="100%" alt="Charles capture nodejs app error">


### 根因分析

Charles抓包拓扑:

<img src="https://gongpengjun.com/imgs/network/charles_packet_capture_nodejs_app.png" alt="Charles capture nodejs app packet">

```
node app.js → Charles Proxy → remote server (jsonplaceholder.typicode.com)
```

查看nodejs应用控制台报错输出：

```
{
  error: Error: self signed certificate in certificate chain
      at TLSSocket.onConnectSecure (_tls_wrap.js:1501:34)
      at TLSSocket.emit (events.js:315:20)
      at TLSSocket.EventEmitter.emit (domain.js:483:12)
      at TLSSocket._finishInit (_tls_wrap.js:936:8)
      at TLSWrap.ssl.onhandshakedone (_tls_wrap.js:710:12) {
    code: 'SELF_SIGNED_CERT_IN_CHAIN'
}
```

根据错误码 SELF_SIGNED_CERT_IN_CHAIN 查找nodejs文档：

https://nodejs.org/dist/latest-v18.x/docs/api/tls.html

- `'SELF_SIGNED_CERT_IN_CHAIN'`: Self signed certificate in certificate chain.

说明nodejs通过charles获得一个charles的自签名证书，nodejs不认可，于是报错。

### 解决方案

关闭Node TLS对证书的校验：

```shell
$ export NODE_TLS_REJECT_UNAUTHORIZED='0'
```

or

```shell
$ NODE_TLS_REJECT_UNAUTHORIZED='0' node app.js
```

### 最佳实践

```shell
$ export http_proxy=http://127.0.0.1:8888
$ export https_proxy=http://127.0.0.1:8888
$ export NODE_TLS_REJECT_UNAUTHORIZED='0'
$ node app.js
```


### 参考资料

- https://nodejs.org/en/knowledge/HTTP/clients/how-to-create-a-HTTP-request/
- https://nodejs.org/dist/latest-v18.x/docs/api/tls.html
- https://stackoverflow.com/a/45088585/328435
