---
layout: post
title: Thrift 0.9.1安装步骤 - Mac OS X 10.14
date:   2018-12-13 17:00:00
categories: C++
---

//# Thrift 0.9.1安装步骤 - Mac OS X 10.14

### 系统环境

Sytem Config  | Version       | Path
------------- | ------------- | -------------
Mac OS X | Mojave 10.14.1 (18B75) |
Xcode  | Version 10.1 (10B61) | /Applications/Xcode.app
javac  | 10.0.2 | /usr/bin/javac

### 0. 卸载Thrift 0.9.3

```shell
$ which thrift
/usr/local/opt/thrift@0.9/bin/thrift
$ brew list|grep thrift
thrift@0.9
$ brew uninstall thrift@0.9
```

### 1. 安装Boost

```shell
$ wget https://dl.bintray.com/boostorg/release/1.69.0/source/boost_1_69_0.tar.gz
$ tar zxvf boost_1_69_0.tar.gz
$ cd boost_1_69_0
$ ./bootstrap.sh
$ sudo ./b2 threading=multi address-model=64 variant=release stage install
```

### 2. 安装libevent

```shell
$ wget https://github.com/libevent/libevent/releases/download/release-2.1.8-stable/libevent-2.1.8-stable.tar.gz
$ tar zxvf libevent-2.1.8-stable.tar.gz
$ cd libevent-2.1.8-stable
$ ./configure --prefix=/usr/local 
$ make
$ sudo make install
```

### 3. 安装libtool

```
$ wget http://mirrors.kernel.org/gnu/libtool/libtool-2.2.6b.tar.gz
$ tar xzvf libtool-2.2.6b.tar.gz
$ cd libtool-2.2.6b
$ ./configure --prefix=/usr/local 
$ make
$ sudo make install
```

```shell
$ wget http://mirrors.kernel.org/gnu/automake/automake-1.11.tar.gz
$ tar xzvf automake-1.11.tar.gz
$ cd automake-1.11
$ ./configure --prefix=/usr/local
$ make
$ sudo make install
```

### 4. 安装Thrift 0.9.1

```shell
$ wget https://github.com/apache/thrift/archive/0.9.1.zip
$ unzip 0.9.1.zip
$ cd thrift-0.9.1
$ ./bootstrap.sh
$ export CXXFLAGS="-std=c++11"
$ ./configure --prefix=/usr/local/thrift-0.9.1 --without-java --without-ruby --without-haskell --without-erlang --without-python --without-perl
$ make CXXFLAGS=-stdlib=libstdc++
$ sudo make install
```

```shell
$ git clone https://github.com/apache/thrift.git
$ git checkout 0.9.1
$ ./bootstrap.sh
$ ./configure --prefix=/usr/local/ --with-boost=/usr/local --with-libevent=/usr/local
$ make
$ sudo make install
```

### 参考链接

- [Apache Thrift OS X Setup](http://thrift.apache.org/docs/install/os_x)
- [MAC OS X 10.9 使用Thrift 0.9.1](http://www.apmbe.com/mac-os-x-10-9-%E5%AE%89%E8%A3%85-thrift-0-9-1/)
