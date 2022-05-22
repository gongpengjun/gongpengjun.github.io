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

本文就针对使用较多的Java 8 + Spring 5 + Jetty 9组合进行源码分析，看看使用服务是怎么启动，怎么运行的。

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

小结：Servlet Spec约定的服务模型

<img src="https://gongpengjun.com/imgs/java_servlet_container_and_servlet.svg" width="100%" alt="Java Servlet Container and Servlet">

### 1.2、概念解释

- Web Server: 监听TCP端口直接响应用户HTTP请求的服务器，Apache、Nginx是通用的Web Server，Tomcat、Jetty也都实现了Web Server的功能。Spring Boot中用[WebServer](https://docs.spring.io/spring-boot/docs/2.3.6.RELEASE/api/org/springframework/boot/web/server/WebServer.html)接口表示Web Server。
- Web Container: 即Servlet Container，从Web Server接收请求并路由给Servlet处理后将结果返回给Web Server，可以独立部署或也可内嵌在Web Server中。Servlet Spec中以[ServletContext](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/ServletContext.java)代表ServletContainer。
- Spring  MVC：即Spring Web MVC，Spring中的类[DispatcherServlet](https://docs.spring.io/spring-framework/docs/5.2.11.RELEASE/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)实现了Servlet接口，从Web Container接收请求，转发给用户定义的Controller处理。
- Web Application: 遵循Spring框架的MVC机制提供Controller实现业务逻辑网络应用。其实也可以不用Spring直接遵循Servlet接口实现业务逻辑。

## 2、Spring Boot启动过程

了解了Servlet规范后，我们来看Spring的总体结构就很容易理解了：Spring Boot自动集成了Spring Framework、Jetty，其中Jetty既扮演Web Server的角色，又扮演Servlet Container的角色，Spring Framework则提供DispatcherServlet来衔接Jetty Servlet Container和应用开发者实现的Controllers。

### 2.1、Spring Boot 启动 Jetty Server

Java应用入口函数`main()`调用`SpringApplication.run()`来启动服务。

```java
@SpringBootApplication
@EnableFeignClients
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class);
  }
}
```

1️⃣ `SpringApplication.run()`会创建AnnotationConfigServletWebServerApplicationContext对象并调用其`refresh()`方法

```java
package org.springframework.boot;
public class SpringApplication {
  public static final String DEFAULT_SERVLET_WEB_CONTEXT_CLASS = "org.springframework.boot."
                 + "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";
  public ConfigurableApplicationContext run(String... args) {
    ConfigurableApplicationContext context = createApplicationContext();
    refreshContext(context);
    return context;
  }
  
  protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
    return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
  }
  private void refreshContext(ConfigurableApplicationContext context) {
    context.refresh();
    // AnnotationConfigServletWebServerApplicationContext.refresh()
  }
}
```

2️⃣ `AbstractApplicationContext`实现了`refresh()`方法，这是Spring Framework的核心方法。

```java
package org.springframework.context.support;
public abstract class AbstractApplicationContext extends DefaultResourceLoader 
                                     implements ConfigurableApplicationContext {
  @Override
  public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
      // Prepare this context for refreshing.
      prepareRefresh();

      // Tell the subclass to refresh the internal bean factory.
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

      // Prepare the bean factory for use in this context.
      prepareBeanFactory(beanFactory);

      // Allows post-processing of the bean factory in context subclasses.
      postProcessBeanFactory(beanFactory);

      // Invoke factory processors registered as beans in the context.
      invokeBeanFactoryPostProcessors(beanFactory);

      // Register bean processors that intercept bean creation.
      registerBeanPostProcessors(beanFactory);

      // Initialize message source for this context.
      initMessageSource();

      // Initialize event multicaster for this context.
      initApplicationEventMulticaster();

      // Initialize other special beans in specific context subclasses.
      onRefresh();

      // Check for listener beans and register them.
      registerListeners();

      // Instantiate all remaining (non-lazy-init) singletons.
      finishBeanFactoryInitialization(beanFactory);

      // Last step: publish corresponding event.
      finishRefresh();
    }
  }
}
```

- `prepareRefresh()`准备和验证系统属性和环境变量
- `obtainFreshBeanFactory()`创建Bean Factory并扫描加载所有的Bean Definition
- `prepareBeanFactory()` 配置Bean Factory的功能，处理特殊处理规则和内置Bean
- `postProcessBeanFactory()`预留给子类的扩展点
- `invokeBeanFactoryPostProcessors()`调用Bean Factory后处理器
- `registerBeanPostProcessors()`注册Bean后处理器
- `initMessageSource()`初始化Message Source，支持多语言
- `initApplicationEventMulticaster()`初始化Spring Event Pub/Sub机制
- `onRefresh()`预留给子类初始化特殊的Bean，ServletWebServerApplicationContext子类会在此初始化WebServer
- `registerListeners()`注册Spring Event Pub/Sub机制中的Listeners
- `finishBeanFactoryInitialization()`实例化所有的单例Bean(标记为懒加载的单例Bean除外)
- `finishRefresh()`发布`ContextRefreshedEvent`事件通知此ApplicationContext初始化并refresh完成

3️⃣ 子类ServletWebServerApplicationContext的`onRefresh()`在refresh过程中被调用，onRefresh()中创建嵌入式WebServer。

```java
package org.springframework.boot.web.servlet.context;
public class ServletWebServerApplicationContext extends GenericWebApplicationContext 
                                  implements ConfigurableWebServerApplicationContext {
  @Override
  protected void onRefresh() {
    super.onRefresh();
    createWebServer();
  }
}
```

4️⃣ ServletWebServerApplicationContext的`createWebServer()`会查找注册的ServletWebServerFactory并创建WebServer

```java
package org.springframework.boot.web.servlet.context;
public class ServletWebServerApplicationContext extends GenericWebApplicationContext 
                                  implements ConfigurableWebServerApplicationContext {
  private void createWebServer() {
    ServletWebServerFactory factory = getWebServerFactory();
    this.webServer = factory.getWebServer(getSelfInitializer());
  }
}
```

Spring Boot内置了三个硬编码注册的ServletWebServerFactory，分别是EmbeddedTomcat、EmbeddedJetty、EmbeddedUndertow，根据classpath的jar配置，这三个ServletWebServerFactory只有一个会启用，我们以EmbeddedJetty为例，如果classpath中找到jetty特有的`WebAppContext.class`的话，则会注册[JettyServletWebServerFactory](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/ServletWebServerFactoryConfiguration.java) Bean。

```java
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass({ Servlet.class, Server.class, Loader.class, WebAppContext.class })
@ConditionalOnMissingBean(value = ServletWebServerFactory.class, search = SearchStrategy.CURRENT)
static class EmbeddedJetty {
  @Bean
  JettyServletWebServerFactory JettyServletWebServerFactory() {
  	return new JettyServletWebServerFactory();;
  }
}
```

`getSelfInitializer()`把`ServletWebServerApplicationContext::selfInitialize`方法转化为ServletContextInitializer lamda类型返回

```java
package org.springframework.boot.web.servlet.context;
public class ServletWebServerApplicationContext extends GenericWebApplicationContext 
                                  implements ConfigurableWebServerApplicationContext {
  private org.springframework.boot.web.servlet.ServletContextInitializer getSelfInitializer() {
    return this::selfInitialize;
  }
}
```

于是以`ServletWebServerApplicationContext::selfInitialize`为参数调用`JettyServletWebServerFactory.getWebServer()`

```java
package org.springframework.boot.web.embedded.jetty;
public class JettyServletWebServerFactory extends AbstractServletWebServerFactory
		implements ConfigurableJettyWebServerFactory, ResourceLoaderAware {
  @Override
  public WebServer getWebServer(ServletContextInitializer... initializers) {
    JettyEmbeddedWebAppContext context = new JettyEmbeddedWebAppContext();
    int port = Math.max(getPort(), 0);
    InetSocketAddress address = new InetSocketAddress(getAddress(), port);
    Server server = createServer(address);
    configureWebAppContext(context, initializers);
    server.setHandler(addHandlerWrappers(context));
    this.logger.info("Server initialized with port: " + port);
    return getJettyWebServer(server);
  }

   private Server createServer(InetSocketAddress address) {
     Server server = new Server(getThreadPool());
     server.setConnectors(new Connector[] { createConnector(address, server) });
     server.setStopTimeout(0);
     return server;
   }
}
```

Jetty的`Server`对象管理Web Server，负责监听端口，接收用户请求，此处只是创建`Server`对象，并未开始端口监听。

`JettyEmbeddedWebAppContext`继承自Jetty的`WebAppContext`，角色是Servlet Container，其内部类`WebAppContext.Context`实现了`ServletContext`接口。

`JettyServletWebServerFactory.getWebServer()`先将JettyEmbeddedWebAppContext设置为Jetty `Server`对象的handler，这样就完成了Jetty Web Server和Jetty Web Container之间的关联。

### 2.2、Spring Boot 注册DispatcherServlet

- - -

## 3、结论

1. 
1. 
1. 

## 4、参考

- Servlet History: https://jamesgdriscoll.wordpress.com/2010/02/09/servlet-history/
- Java Servlet 3.1 Specification: https://jcp.org/en/jsr/detail?id=340
- Java Servlet 4.0 Specification: https://jcp.org/en/jsr/detail?id=369
- Understanding Java Servlet Architecture: https://codeburst.io/understanding-java-servlet-architecture-b74f5ea64bf4

