---
title: Spring启动过程分析
date: 2017-06-25 19:56:24
tags: 
    - Spring
---

## Spring简介

Spring的最基本的功能就是创建对象及管理这些对象之间的依赖关系，实现低耦合、高内聚。

还提供像通用日志记录、性能统计、安全控制、异常处理等面向切面的能力，还能帮我们管理最头疼的数据库事务，本身提供了一套简单的JDBC访问实现，提供与第三方数据访问框架集成（如hibernate、JPA、Mybatis），

与各种Java EE技术整合（如Java Mail、任务调度等等），提供一套自己的web层框架Spring MVC、而且还能非常简单的与第三方web框架集成。

从这里我们可以认为Spring是一个超级粘合平台，除了自己提供功能外，还提供粘合其他技术和框架的能力，

从而使我们可以更自由的选择到底使用什么技术进行开发。而且不管是JAVA SE（C/S架构）应用程序还是JAVA EE（B/S架构）应用程序都可以使用这个平台进行开发。

## Spring运行流程

Spring的启动过程其实就是其IOC容器的启动过程，对于web程序，IOC容器启动过程即是建立上下文的过程。

### web.xml中的配置

```yaml
    <servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.Springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
    </context-param>

    <listener>      
      <listener-class>org.Springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
```

### 分析
```yaml
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/mvc-dispatcher-servlet.xml</param-value>
</context-param>

<listener>      
  <listener-class>org.Springframework.web.context.ContextLoaderListener</listener-class>
 </listener>
```

这段是加载Spring配置文件，初始化上下文，ContextLoaderListener是一个实现了ServletContextListener接口的监听器，

在启动项目时会触发contextInitialized方法（该方法主要完成ApplicationContext对象的创建），在关闭项目时会触发contextDestroyed方法（该方法会执行ApplicationContext清理操作）


①启动项目时触发contextInitialized方法，该方法就做一件事：通过父类contextLoader的initWebApplicationContext方法创建Spring上下文对象。

②initWebApplicationContext方法做了三件事：创建 WebApplicationContext；加载对应的Spring文件创建里面的Bean实例；将WebApplicationContext放入 ServletContext（就是Java Web的全局变量）中。

③createWebApplicationContext创建上下文对象，支持用户自定义的 上下文对象，但必须继承自ConfigurableWebApplicationContext，而Spring MVC默认使用ConfigurableWebApplicationContext作为ApplicationContext（它仅仅是一个接口）的实 现。

④configureAndRefreshWebApplicationContext方法用 于封装ApplicationContext数据并且初始化所有相关Bean对象。它会从web.xml中读取名为 contextConfigLocation的配置，这就是Spring xml数据源设置，然后放到ApplicationContext中，最后调用传说中的refresh方法执行所有Java对象的创建。

⑤完成ApplicationContext创建之后就是将其放入ServletContext中，注意它存储的key值常量。

```yaml
    <servlet>
        <servlet-name>mvc-dispatcher</servlet-name>
        <servlet-class>org.Springframework.web.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>
```

这段为启动初始化DispatcherServlet ，web.xml中设置了Servlet的load-on-startup：表示启动容器时初始化该Servlet。

url-pattern：表示哪些请求交给Spring Web MVC处理， “/” 是用来定义默认servlet映射的。也可以如“*.html”表示拦截所有以html为扩展名的请求。

DispatcherServlet默认使用WebApplicationContext（ContextLoaderListener初始化产生）作为上下文，Spring默认配置文件为“/WEB-INF/[servlet名字]-servlet.xml”**（该名字可以自定义，在<param-value>）**。

覆盖配置文件举例：
```yaml
<servlet>
    <servlet-name>chapter2</servlet-name>
    <servlet-class>org.Springframework.web.servlet.DispatcherServlet</servlet-class>
    <load-on-startup>1</load-on-startup>

    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:Spring-servlet-config.xml</param-value>
    </init-param>
</servlet>
```
如果使用如上配置，Spring Web MVC框架将加载“classpath:Spring-servlet-config.xml”来进行初始化上下文而不是“/WEB-INF/[servlet名字]-servlet.xml”

## DispatcherServlet初始化顺序

（1）HttpServletBean继承HttpServlet，因此在Web容器启动时将调用它的init方法，该初始化方法的主要作用：将Servlet初始化参数（init-param）设置到该组件上（如contextAttribute、contextClass、namespace、contextConfigLocation），通过BeanWrapper简化设值过程，方便后续使用；提供给子类初始化扩展点，initServletBean()，该方法由FrameworkServlet覆盖。

（2）FrameworkServlet继承HttpServletBean，通过initServletBean()进行Web上下文初始化，该方法主要覆盖一下两件事情：初始化web上下文；提供给子类初始化扩展点。

（3）DispatcherServlet继承FrameworkServlet，并实现了onRefresh()方法提供一些前端控制器相关的配置。

整个DispatcherServlet初始化的过程和做了些什么事情，具体主要做了如下两件事情：

1、初始化Spring Web MVC使用的Web上下文，并且指定父容器为WebApplicationContext（ContextLoaderListener加载了的根上下文）；

2、初始化DispatcherServlet使用的策略，如HandlerMapping、HandlerAdapter等。