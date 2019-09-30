---
title: Spring MVC常见配置举例
date: 2017-06-21 19:56:24
tags: 
    - Spring MVC
---

Spring MVC项目中通常会有二个配置文件，sprng-servlet.xml和applicationContext.xml二个配置文件。

前者主要是用来实现mvc的，后者主要是用来实现ioc的，后者的配置文件中通常有一下几个节点：

## <context:annotation-config />

```yaml
它的作用是隐式地向 Spring 容器注册  AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor、PersistenceAnnotationBeanPostProcessor、RequiredAnnotationBeanPostProcessor 这4个BeanPostProcessor。其作用是如果你想在程序中使用注解，就必须先注册该注解对应的类，如下图所示：
```

依赖的类	| 注解
---|---
CommonAnnotationBeanPostProcessor	| @Resource 、@PostConstruct、@PreDestroy
PersistenceAnnotationBeanPostProcessor的Bean	| @PersistenceContext
AutowiredAnnotationBeanPostProcessor Bean	| @Autowired  @Value
RequiredAnnotationBeanPostProcessor | 	@Required
ConfigurationClassPostProcessor|@Configuration
EventListenerMethodProcessor|@EventListener

当然也可以自己进行注册：

```java
<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor "/> 
<bean class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/> 
```
## <context:component-scan base-package="com.*" >

<context:component-scan/> 配置项不但启用了对类包进行扫描以实施注释驱动 Bean 定义的功能，同时还启用了注释驱动自动注入的功能（即还隐式地在内部注册了 AutowiredAnnotationBeanPostProcessor 和 CommonAnnotationBeanPostProcessor），
因此当使用 <context:component-scan/> 后，就可以将 <context:annotation-config/> 移除了。

 在这里有一个比较有意思的问题，就是扫描是否需要在二个配置文件都配置一遍，我做了这么几种测试：
 
 （1）只在applicationContext.xml中配置如下:
 
   <context:component-scan base-package="com.login" />

```yaml
扫描到有@RestController、@Controller、@Compoment、@Repository等注解的类，则把这些类注册为Bean。　　
启动正常，但是任何请求都不会被拦截，简而言之就是@Controller失效。
```
 （2）只在spring-servlet.xml中配置上述配置
 
```yaml
启动正常，请求也正常，但是事物失效，也就是不能进行回滚
```
 （3）在applicationContext.xml和spring-servlet.xml中都配置上述信息
 ```yaml
启动正常，请求正常，也是事物失效，不能进行回滚
```
 (4）在applicationContext.xml中配置如下
<context:component-scan base-package="com.login" />

在spring-servlet.xml中配置如下
<context:component-scan base-package="com.sohu.login.web" />
 
```yaml
此时启动正常，请求正常，事物也正常了。
```

**结论：**
```yaml
在spring-servlet.xml中只需要扫描所有带@Controller注解的类，在applicationContext中可以扫描所有其他带有注解的类（也可以过滤掉带@Controller注解的类）。
```

## <mvc:annotation-driven />

它会自动注册RequestMappingHandlerMapping 与RequestMappingHandlerAdapter （ 3.1 以后DefaultAnnotationHandlerMapping 和AnnotationMethodHandlerAdapter已经可以不用了）

## spring中classpath和classpath*的配置区别

我们以spring中加载properties资源文件为例讲解，目录结构大致如下：

```yaml
src
├─main
│  ├─filters
│  │
│  ├─java
│  │  └─com
│  │      └─micmiu
│  │          ├─demoweb
│  │          │  │ ....
│  │          │  │
│  │          │  └─utils
│  │          │
│  │          └─modules
│  │
│  ├─resources
│  │  │  application.properties
│  │  │  applicationContext-shiro.xml
│  │  │  applicationContext.xml
│  │  │  hibernate.cfg.xml
│  │  │  log4j.properties
│  │  │  spring-mvc.xml
│  │  │  spring-view.xml
│  │
│  └─webapp
│      │
│      └─WEB-INF
│
└─test
    ├─java
    │  └─com
    │      └─micmiu
    │          ├─demoweb
    │          │      TestOther.java
    │
    └─resources
            application.properties
```

同时 在该项目的lib中添加一个测试的micmiu-test.jar包，jar包中的文件结构如下：

```yaml
micmiu-test.jar
│  application.properties
│
├─com
│  └─micmiu
│      └─test
│              application.properties
│              RunApp.class
│
└─META-INF
        MANIFEST.MF
```

从准备的测试环境中我们可以看到在不同目录下的四个同名的application.properties资源文件。

测试代码：TestClassPath.java  

```java
package com.micmiu.demoweb;
 
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
 
public class TestClassPath {
　　public static void main(String[] args) {
　　　　ApplicationContext ctx = new ClassPathXmlApplicationContext("classpath:/applicationContext.xml");
　　　　　　System.out.println(ctx.getClassLoader().getResource("").getPath());
 　　}
}
```
测试结果:

spring配置文件：applicationContext.xml 中两种不同的properties文件加载配置：

第一种配置：classpath:

<context:property-placeholder ignore-unresolvable="true" location="classpath:/application.properties" />
 
输出：
Loading properties file from class path resource [application.properties]

第二种配置: classpath*:

<context:property-placeholder ignore-unresolvable="true"  location="classpath*:/application.properties" />

输出：
Loading properties file from URL [file:/D:/workspace_sun/framework-dev/micmiu-demoweb/target/test-classes/application.properties]
Loading properties file from URL [file:/D:/workspace_sun/framework-dev/micmiu-demoweb/target/classes/application.properties]
Loading properties file from URL [jar:file:/D:/micmiu-test.jar!/application.properties]

**结论：**

同名资源存在时，classpath: 只从第一个符合条件的classpath中加载资源，而classpath*: 会从所有的classpath中加载符合条件的资源

classpath*:需要遍历所有的classpath，效率肯定比不上classpath，因此在项目设计的初期就尽量规划好资源文件所在的路径，避免使用classpath*来加载