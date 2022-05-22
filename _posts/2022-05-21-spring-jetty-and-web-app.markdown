---
layout: post
title: Spring Jetty And Web App
date:   2022-05-21 21:19:00
categories: Java Framework
---

Java服务端最流行的框架是[Spring](https://spring.io/)，框架越成熟，业务开发时越能聚焦业务，这很省心、也很有效率。

直到某一天遇到一个错误`IllegalStateException: AnnotationConfigServletWebServerApplicationContext has not been refreshed yet`，要想理解这个错误的根因并找到解决办法，就需要深入理解Spring的工作原理和实现细节了。

本文就针对使用最多的Java 8 + Spring 5.x + Jetty 9.x组合，进行源码分析，看看使用Spring框架的Java服务是怎么启动和运行的。

- - -

## 1、结论 - Spring + Jetty + Web App一览

![Spring Jetty And Web App](/imgs/spring_jetty_web_app.svg)

一图胜千言，下文这幅概览图的详细解释。
