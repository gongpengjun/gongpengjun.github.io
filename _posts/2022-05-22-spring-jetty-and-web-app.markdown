---
layout: post
title: Spring Jetty And Web App
date:   2022-05-22 23:44:00
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

一图胜千言，下文对这幅一览图进行详细解释。

### 1.1、Java服务端标准 - Servlet

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

一个实现了Servlet接口的类就是一个Servlet，通过`init()`初始化，通过`destroy()`销毁，在存活期间通过`service()`方法处理请求。

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

备注：[Java Servlet 3.1 Specification](https://jcp.org/en/jsr/detail?id=340) 2013年发布，Spring 5只支持3.0以上的Servlet规范，[Java Servlet 4.0 Specification](https://jcp.org/en/jsr/detail?id=369) 于2017年发布，主要增加了http2.0的支持。

### 1.2、概念解释

- Web Server: 监听TCP端口直接响应用户HTTP请求的服务器，Apache、Nginx是通用的Web Server，Tomcat、Jetty也都实现了Web Server的功能。Spring Boot中用[WebServer](https://docs.spring.io/spring-boot/docs/2.3.6.RELEASE/api/org/springframework/boot/web/server/WebServer.html)接口表示Web Server。
- Web Container: 即Servlet Container，从Web Server接收请求并路由给Servlet处理后将结果返回给Web Server，可以独立部署或也可内嵌在Web Server中。Servlet Spec中以[ServletContext](https://github.com/javaee/servlet-spec/blob/master/src/main/java/javax/servlet/ServletContext.java)代表ServletContainer。
- Spring  MVC：即Spring Web MVC，Spring中的类[DispatcherServlet](https://docs.spring.io/spring-framework/docs/5.2.11.RELEASE/javadoc-api/org/springframework/web/servlet/DispatcherServlet.html)实现了Servlet接口，从Web Container接收请求，转发给用户定义的Controller处理。
- Web Application: 遵循Spring框架的MVC机制提供Controller实现业务逻辑网络应用。其实也可以不用Spring直接遵循Servlet接口实现业务逻辑。

## 2、Spring Boot启动过程

了解了Servlet规范后，我们来看Spring的总体结构就很容易理解了：Spring Boot自动集成了Spring Framework、Jetty，其中Jetty既扮演Web Server的角色，又扮演Servlet Container的角色，Spring Framework则提供DispatcherServlet来衔接Jetty Servlet Container和应用开发者实现的Controllers。

### 2.1、Spring Boot 初始化 Jetty Server 和 Jetty Servlet Container

1️⃣ Java应用入口函数`main()`调用`SpringApplication.run()`来启动服务。

```java
@SpringBootApplication
@EnableFeignClients
public class Application {
  public static void main(String[] args) {
    SpringApplication.run(Application.class);
  }
}
```

2️⃣ `SpringApplication.run()`会创建AnnotationConfigServletWebServerApplicationContext对象并调用其`refresh()`方法

```java
package org.springframework.boot;
public class SpringApplication {
  public ConfigurableApplicationContext run(String... args) {
    ConfigurableApplicationContext context = createApplicationContext();
    refreshContext(context);
    return context;
  }
  protected ConfigurableApplicationContext createApplicationContext() {
    Class<?> contextClass = Class.forName("AnnotationConfigServletWebServerApplicationContext");
    return (ConfigurableApplicationContext)BeanUtils.instantiateClass(contextClass);
  }
  private void refreshContext(ConfigurableApplicationContext context) {
    context.refresh();
    // AnnotationConfigServletWebServerApplicationContext.refresh()
  }
}
```

3️⃣  `AbstractApplicationContext`实现了`refresh()`方法，这是Spring Framework的核心方法，注释说明每一步的作用。

```java
package org.springframework.context.support;
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {
  @Override
  public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
      // 准备和验证系统属性和环境变量
      prepareRefresh();
      // 创建Bean Factory并扫描加载所有的Bean Definition
      ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
      // 配置Bean Factory的功能，处理特殊处理规则和内置Bean
      prepareBeanFactory(beanFactory);
      // 预留给子类的扩展点
      postProcessBeanFactory(beanFactory);
      // 调用Bean Factory后处理器
      invokeBeanFactoryPostProcessors(beanFactory);
      // 注册Bean后处理器
      registerBeanPostProcessors(beanFactory);
      // 初始化Message Source，支持多语言
      initMessageSource();
      // 初始化ApplicationEventMulticaster, 缺省为SimpleApplicationEventMulticaster
      initApplicationEventMulticaster();
      // 预留给子类的扩展点
      onRefresh();
      // 注册实现ApplicationListener接口的Bean为Listener
      registerListeners();
      // 实例化所有的单例Bean(标记为懒加载的单例Bean除外)
      finishBeanFactoryInitialization(beanFactory);
      // 注册DefaultLifecycleProcessor并调用onRefresh方法
      finishRefresh();
    }
  }
}
```

其中关键点：

- `onRefresh()` 子类ServletWebServerApplicationContext重载来创建WebServer
- `finishRefresh()` 调用`DefaultLifecycleProcessor.onRefresh()`来扫描所有实现LifeCycle接口的Bean并调用其`start()`方法

4️⃣ 子类ServletWebServerApplicationContext的`onRefresh()`在refresh过程中被调用，onRefresh()中创建嵌入式WebServer。

```java
package org.springframework.boot.web.servlet.context;
public class ServletWebServerApplicationContext extends GenericWebApplicationContext implements ConfigurableWebServerApplicationContext {
  @Override
  protected void onRefresh() {
    super.onRefresh();
    createWebServer();
  }
}
```

5️⃣ ServletWebServerApplicationContext的`createWebServer()`会查找注册的ServletWebServerFactory并创建WebServer

```java
package org.springframework.boot.web.servlet.context;
public class ServletWebServerApplicationContext extends GenericWebApplicationContext implements ConfigurableWebServerApplicationContext {
  private void createWebServer() {
    ServletWebServerFactory factory = getWebServerFactory();
    this.webServer = factory.getWebServer(getSelfInitializer());
    this.getBeanFactory().registerSingleton("webServerStartStop", 
       new WebServerStartStopLifecycle(this, this.webServer));
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
public class ServletWebServerApplicationContext extends GenericWebApplicationContext  implements ConfigurableWebServerApplicationContext {
  private org.springframework.boot.web.servlet.ServletContextInitializer getSelfInitializer() {
    return this::selfInitialize;
  }
}
```

于是以`ServletWebServerApplicationContext::selfInitialize`为参数调用`JettyServletWebServerFactory.getWebServer()`

```java
package org.springframework.boot.web.embedded.jetty;
public class JettyServletWebServerFactory extends AbstractServletWebServerFactory implements ConfigurableJettyWebServerFactory, ResourceLoaderAware {
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

注意：`configureWebAppContext()`会将`ServletWebServerApplicationContext::selfInitialize`包装为`ServletContextInitializerConfiguration` 然后作为配置添加到`JettyEmbeddedWebAppContext` 的父类`WebAppContext`的`_configurations`变量中。

6️⃣ `getJettyWebServer()`会创建`JettyWebServer`对象并调用其`initialize()`方法进行初始化

```java
package org.springframework.boot.web.embedded.jetty;
public class JettyServletWebServerFactory {
  protected JettyWebServer getJettyWebServer(Server server) {
    return new JettyWebServer(server, getPort() >= 0);
  }
}
```

JettyWebServer的`initialize()`方法中，为了避免Jetty的`Server`在`start()`时开始端口监听，先将其connectors清空后，然后再调用Jetty`Server.start()`方法，这样ServletContext即`WebAppContext.Context`就可用了。

```java
package org.springframework.boot.web.embedded.jetty;
public class JettyWebServer implements WebServer {
  public JettyWebServer(Server server, boolean autoStart) {
    this.autoStart = autoStart;
    this.server = server;
    initialize();
  }

  private void initialize() {
    // Cache the connectors and then remove them to prevent requests being
    // handled before the application context is ready.
    this.connectors = this.server.getConnectors();
    this.server.addBean(new AbstractLifeCycle() {
      @Override
      protected void doStart() throws Exception {
      	JettyWebServer.this.server.setConnectors(null);
      }
    });
    // Start the server so that the ServletContext is available
    this.server.start();
  }
}
```

Jetty`Server.start()`方法中会触发Servlet Container `WebAppContext`的`doStart()`并执行其的`_configurations`变量中的配置对象的`configure()`方法。

7️⃣ `ServletContextInitializerConfiguration.configure`其实就是调用`callInitializers()`，会逐个调用注册的`ServletContextInitializer` lamda，其中就有`ServletWebServerApplicationContext::selfInitialize`方法。

```java
package org.springframework.boot.web.embedded.jetty;
public class ServletContextInitializerConfiguration extends AbstractConfiguration {
  private final ServletContextInitializer[] initializers;
  @Override
  public void configure(WebAppContext context) throws Exception {
    callInitializers(context);
  }
  private void callInitializers(WebAppContext context) throws ServletException {
    for (ServletContextInitializer initializer : this.initializers) {
    	initializer.onStartup(context.getServletContext());
    }
  }
}
```

8️⃣ `ServletWebServerApplicationContext::selfInitialize`创建ServletContextInitializerBeans并调用其`onStartup()`方法 

```java
package org.springframework.boot.web.servlet.context;
public class ServletWebServerApplicationContext extends GenericWebApplicationContext 
                                  implements ConfigurableWebServerApplicationContext {
  private void selfInitialize(ServletContext servletContext) {
    for (ServletContextInitializer beans : getServletContextInitializerBeans()) {
      beans.onStartup(servletContext);
    }
  }
  Collection<ServletContextInitializer> getServletContextInitializerBeans() {
    return new ServletContextInitializerBeans(getBeanFactory());
  }
}
```

ServletContextInitializerBeans会从spring container中查找所有的ServletContextInitializer接口实现类并调用其`onStartup()`方法。

```java
public class ServletContextInitializerBeans extends AbstractCollection<ServletContextInitializer> {
  @SafeVarargs
  public ServletContextInitializerBeans(ListableBeanFactory beanFactory, 
    Class<? extends ServletContextInitializer>... initializerTypes) {
    this.initializers = new LinkedMultiValueMap<>();
    this.initializerTypes = (initializerTypes.length != 0) 
                  ? Arrays.asList(initializerTypes) 
                  : Collections.singletonList(ServletContextInitializer.class);
    addServletContextInitializerBeans(beanFactory);
  }
  private void addServletContextInitializerBeans(ListableBeanFactory beanFactory) {
    for (Class<? extends ServletContextInitializer> initializerType : this.initializerTypes) {
      for (Entry<String, ? extends ServletContextInitializer> initializerBean : 
                                         getOrderedBeansOfType(beanFactory, initializerType)) {
        addServletContextInitializerBean(initializerBean.getKey(), initializerBean.getValue(), beanFactory);
      }
    }
  }
  // type = ServletContextInitializer.class
  private <T> List<Entry<String, T>> getOrderedBeansOfType(
    ListableBeanFactory beanFactory, Class<T> type, Set<?> excludes) {
    String[] names = beanFactory.getBeanNamesForType(type, true, false);
    //返回 names = String[]{"dispatcherServletRegistration"}
    // DispatcherServletRegistrationBean 由 DispatcherServletRegistrationConfiguration 配置
    Map<String, T> map = new LinkedHashMap<>();
    for (String name : names) {
      if (!excludes.contains(name) && !ScopedProxyUtils.isScopedTarget(name)) {
        T bean = beanFactory.getBean(name, type);
        if (!excludes.contains(bean)) {
          map.put(name, bean);
        }
      }
    }
    return beans;
  }
}
```

9️⃣ DispatcherServletRegistrationBean实现了ServletContextInitializer接口，负责将DispatcherServlet注册为Jetty ServletContainer的根Servlet。

- ServletRegistrationBean是Spring Boot的servlet注册接口，底层调用`ServletContext.addServlet()`进行Servlet注册

```java
package org.springframework.boot.web.servlet;
public class ServletRegistrationBean<T extends Servlet> extends DynamicRegistrationBean<ServletRegistration.Dynamic> {
	private T servlet;
	private Set<String> urlMappings = new LinkedHashSet<>();
	public ServletRegistrationBean(T servlet, boolean alwaysMapUrl, String... urlMappings) {
		this.servlet = servlet;
		this.urlMappings.addAll(Arrays.asList(urlMappings));
	}
	@Override
	protected ServletRegistration.Dynamic addRegistration(String description, ServletContext servletContext) {
		String name = getServletName();
		return servletContext.addServlet(name, this.servlet);
	}
}
```

- DispatcherServletRegistrationBean继承自ServletRegistrationBean，负责DispatcherServlet的注册

```java
package org.springframework.boot.autoconfigure.web.servlet;
public class DispatcherServletRegistrationBean 
             extends ServletRegistrationBean<DispatcherServlet> implements DispatcherServletPath {
    public DispatcherServletRegistrationBean(DispatcherServlet servlet, String path) {
        super(servlet, new String[0]);
        super.addUrlMappings(new String[]{this.getServletUrlMapping()});
    }
}
```

至此：Spring Container、Jetty Web Server、Jetty Servlet Container、DispatcherServlet全部初始化完毕且相互关联在一起 。

### 2.2、Spring Boot 启动 Jetty Server 端口监听并接收请求

AbstractApplicationContext的`refresh()`方法的中间步骤`onRefresh()`触发了ServletWebServerApplicationContext的`createWebServer()`，会向Spring Container中注册了`WebServerStartStopLifecycle`对象

```java
this.getBeanFactory().registerSingleton("webServerStartStop", new WebServerStartStopLifecycle(this, this.webServer));
```

🔟 AbstractApplicationContext的`refresh()`方法的最后一步`finishRefresh()`会初始化DefaultLifecycleProcessor并调用其`onRefresh()`方法

```java
package org.springframework.context.support;
public abstract class AbstractApplicationContext extends DefaultResourceLoader implements ConfigurableApplicationContext {
  protected void finishRefresh() {
    initLifecycleProcessor();
    getLifecycleProcessor().onRefresh();
  }
  protected void initLifecycleProcessor() {
    ConfigurableListableBeanFactory beanFactory = getBeanFactory();
    DefaultLifecycleProcessor defaultProcessor = new DefaultLifecycleProcessor();
    defaultProcessor.setBeanFactory(beanFactory);
    this.lifecycleProcessor = defaultProcessor;
    beanFactory.registerSingleton(LIFECYCLE_PROCESSOR_BEAN_NAME, this.lifecycleProcessor);
  }
  LifecycleProcessor getLifecycleProcessor() {
    return this.lifecycleProcessor;
  }
}
```
DefaultLifecycleProcessor的`onRefresh()`方法收集所有的Lifecycle Bean，包含SmartLifecycle，并按SmartLifecycle中`getPhase()`返回的优先级执行`start()`方法
```java
package org.springframework.context.support;
public class DefaultLifecycleProcessor implements LifecycleProcessor, BeanFactoryAware {
  @Override
  public void onRefresh() {
    startBeans(true);
    this.running = true;
  }

  private void startBeans(boolean autoStartupOnly) {
    Map<String, Lifecycle> lifecycleBeans = getLifecycleBeans();
    Map<Integer, LifecycleGroup> phases = new HashMap<>();
    lifecycleBeans.forEach((beanName, bean) -> {
      if (!autoStartupOnly || (bean instanceof SmartLifecycle && ((SmartLifecycle) bean).isAutoStartup())) {
        int phase = getPhase(bean);
        LifecycleGroup group = phases.get(phase);
        if (group == null) {
          group = new LifecycleGroup(phase, this.timeoutPerShutdownPhase, lifecycleBeans, autoStartupOnly);
          phases.put(phase, group);
        }
        group.add(beanName, bean);
      }
    });
    if (!phases.isEmpty()) {
      List<Integer> keys = new ArrayList<>(phases.keySet());
      Collections.sort(keys);
      for (Integer key : keys) {
        phases.get(key).start();
      }
    }
  }
}
```

于是WebServerStartStopLifecycle实现了SmartLifecycle接口，所以会被查到且其`start()`方法会被调用

```java
package org.springframework.boot.web.servlet.context;
class WebServerStartStopLifecycle implements SmartLifecycle {
  private final WebServer webServer; // JettyWebServer
  WebServerStartStopLifecycle(...,WebServer webServer) {
    this.webServer = webServer;
  }
  @Override
  public void start() {
    this.webServer.start();
  }
}
```

WebServerStartStopLifecycle.webServer是JettyWebServer，于是JettyWebServer的`start()`被执行，开始端口监听

```java
package org.springframework.boot.web.embedded.jetty;
public class JettyWebServer implements WebServer {
  @Override
  public void start() throws WebServerException {
    this.server.setConnectors(this.connectors);
    this.server.start();
    for (Handler handler : this.server.getHandlers()) {
      handleDeferredInitialize(handler);
    }
    Connector[] connectors = this.server.getConnectors();
    for (Connector connector : connectors) {
      try {
          connector.start();
      }
      catch (IOException ex) {
          if (connector instanceof NetworkConnector) {
              PortInUseException.throwIfPortBindingException(ex,
                      () -> connector.getPort());
          }
          throw ex;
      }
    }
    this.started = true;
    logger.info("Jetty started on port(s) " + getActualPortsDescription() + 
            " with context path '" + getContextPath() + "'");
  }
}
```

具体就是：先`this.server.setConnectors(this.connectors);`恢复之前缓存的Jetty `Sever`的Connector，然后逐个启动`Connector.start()`

如果端口监听失败则打印：`Web server failed to start. Port 8080 was already in use.`

如果端口监听成功则打印：`Server initialized with port: 8080`

- - -

## 3、总结

1. Spring Boot通过硬编码的方式扫描Tomcat、Jetty、Undertow并注册相应的实现ServletWebServerFactory接口的类，对应Jetty是JettyServletWebServerFactory。
1. Spring Boot调用`AbstractApplicationContext.refresh()`刷新Spring Application Context
1. `AbstractApplicationContext.refresh()`中的`onRefresh()`调用`JettyServletWebServerFactory.getWebServer()`创建**JettyWebServer**和**JettyEmbeddedWebAppContext**，它们分别管理Jetty Web Server和Jetty Servlet Container。在JettyWebServer的初始化方法 `JettyWebServer.initialize()`中触发ServletContextInitializer即ServletWebServerApplicationContext::selfInitialize来注册**DispatcherServlet**。
1. `AbstractApplicationContext.refresh()`中的`finishRefresh()`查找实现SmartLifeCycle接口的WebServerStartStopLifecycle类完成JettyWebServer的真正启动，即开始端口监听和接收用户请求。

## 4、参考

- Servlet History: https://jamesgdriscoll.wordpress.com/2010/02/09/servlet-history/
- Java Servlet 3.1 Specification: https://jcp.org/en/jsr/detail?id=340
- Java Servlet 4.0 Specification: https://jcp.org/en/jsr/detail?id=369
- Understanding Java Servlet Architecture: https://codeburst.io/understanding-java-servlet-architecture-b74f5ea64bf4

