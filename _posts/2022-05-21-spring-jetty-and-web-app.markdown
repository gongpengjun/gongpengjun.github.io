---
layout: post
title: Spring Jetty And Web App
date:   2022-05-21 21:19:00
categories: Java Framework
---

Java服务端最流行的框架是[Spring Framwork](https://spring.io/)，框架越成熟，业务开发时越能聚焦业务，这很省心、也很有效率。

直到某一天遇到一个错误：
```
java.lang.IllegalStateException: AnnotationConfigServletWebServerApplicationContext has not been refreshed yet
```
要想理解这个错误的根因并找到解决办法，就需要深入理解Spring的工作原理和实现细节了。

本文就针对使用最多的Java 8 + Spring 5 + Jetty 9组合进行源码分析，看看使用服务是怎么启动，怎么运行的。

- - -

## 1、结论 - Spring + Jetty + Web App一览

<img src="https://gongpengjun.com/imgs/spring_jetty_web_app.svg" width="100%" alt="Spring Jetty And Web App">

一图胜千言，下文是这幅概览图的详细解释。

### 1.1、Java服务端标准 - Servlet

[Java Servlet 3.1 Specification](https://jcp.org/en/jsr/detail?id=340) 2013年发布，Spring 5只支持3.0以上的Servlet规范，[Java Servlet 4.0 Specification](https://jcp.org/en/jsr/detail?id=369) 于2017年发布，主要增加了http2.0的支持。

Java Servlet Spec主要是约定了Servlet和Servlet容器之间的交互方式。

主要是Servlet、ServletContext两个接口。

[Servlet](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/Servlet.java)接口：

```java
public interface Servlet {
  public void init(ServletConfig config);
  public void service(ServletRequest,ServletResponse);
  public void destroy();
}
```

一个实现了Servlet接口的类就是一个Servlet，通过`init()`初始化，通过`destroy()`销毁，在存活期间通过`service()`方法的ServletRequest参数接收客户端请求并将通过ServletResponse参数返回响应。

[ServletContext](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/ServletContext.java)接口：
```java
public interface ServletContext {
  public ServletRegistration.Dynamic 
  addServlet(String servletName, Servlet servlet);
}
```

一个Servlet容器一般都要提供一个实现了ServletContext接口的类，通过`addServlet()`，使用者可以注册一个或多个Servlet到Servlet容器里，每个Servlet负责一个或一组URI路径，路径为`/`的Servlet称为根Servlet。

于是Servlet Context和Servlet形成如下的关系来为用户提供服务：

<img src="https://gongpengjun.com/imgs/java_servlet_container_and_servlet.svg" width="100%" alt="Java Servlet Container and Servlet">


### 1.2、Java服务端标准 - Servlet

Spring Framework中的类DispatcherServlet就是实现了Servlet接口，Jetty中的WebAppContext类的内部类WebAppContext.Context实现了ServletContext接口。

### 1.2、Web Server/Web Container/Web App


## 2、Java服务端标准 - Servlet

### 2.1、Java Servlet API

Java服务端的技术标准是Java Servlet API，

```

```

- - -

## 3、结论

1. 
1. 
1. 

## 4、参考

- Servlet History: https://jamesgdriscoll.wordpress.com/2010/02/09/servlet-history/
- Java Servlet 3.1 Specification: https://jcp.org/en/jsr/detail?id=340
- Java Servlet 4.0 Specification: https://jcp.org/en/jsr/detail?id=369

