---
title: Spring Boot监控与管理
date: 2019-01-25 16:56:24
tags: 微服务
---
# 1、前言
在微服务架构中，我们将原本庞大的单体系统拆分成多个提供不同服务的应用。虽然各个应用的内部逻辑因分解而得到简化，但是由于部署应用的数量成倍增长，使得系统的维护复杂度大大提高。

对于运维人员来说，随着应用的不断增多，系统集群中出现故障的频率也变得越来越高，虽然在高可用机制的保护下，个体故障不会影响系统的对外服务，但是这些频繁出现的故障需要被及时发现和处理才能长期保证系统处于健康可用状态，为了能对这些成倍增长的应用做到高效运维，传统的运维方式显然是不合适的，所以我们需要实现一套自动化的监控运维机制。而这套机制的运行基础就是不断的收集各个微服务应用的各项指标情况，并根据这些基础指标来制定监控和预警规则，更进一步甚至做到一些自动化的运维操作等。

为了让运维系统能够获取各个微服务应用的相关指标以实现一些常规操作控制，我们需要一套专门用于植入各个微服务应用的接口供监控系统采集信息。而这些接口往往有很大一部分指标都是类似的，比如环境变量、垃圾收集信息、内存信息、线程池信息等。既然这些信息这样通用，那是否有一个标准化的解决方案嘛？

当我们决定使用Spring Boot来做微服务底层架构时，除了它强大的快速开发功能之外，还因为它在Starter POMs中提供了一个特殊依赖模块**spring-boot-starter-actuator**。引入该模块能够自动为Spring Boot构建的应用提供一系列用于监控的端点。同时，Spring Cloud在实心各个微服务组件的时候，进一步为该模块做了不少拓展，比如，为原生端点增加了更多的指标和度量信息（比如在整合Eureka的时候会为/health端点增加相关的信息），并且根据不同的组件还提供了更多更多有空的端点（比如，为API网关组件Zuul提供了/routes端点来返回路由信息）。

**spring-boot-starter-actuator**模块的实现对于微服务的中小团队来说，可以有效的省去或大大减少监控系统和在采集应用指标时的开发量。当然，它并不是万能的，有时候我们需要简单的拓展来帮助我们实现自身系统个性化的监控需求。所以在本节，将详细介绍关于**spring-boot-starter-actuator**模块的内容，包括原生提供的端点以及一些常用的拓展和配置方式等。

# 2、初识actuator

一句话，**actuator是监控系统健康情况的工具**。

下面，我们通过对“快速入门”小节中实现的Spring Boot应用增加*spring-boot-starter-actuator**模块功能，来对它有一个直观的认识。

## 2.1、actuator基本使用
引入**spring-boot-starter-actuator**模块功能依赖只需要在pom.xml文件中新增一个dependency节点，内容如下：

```
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

```
新增该依赖之后，重启应用，我们可以在控制台启动日志中看到如下图的输出：


```
2019-06-06 08:59:40.826  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/env/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.actuat
or.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EnvironmentMvcEndpoint.value(java.lang.String)
2019-06-06 08:59:40.827  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/env || /env.json],methods=[GET],produces=[application/vnd.spring-boot.act
uator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2019-06-06 08:59:40.829  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/loggers/{name:.*}],methods=[GET],produces=[application/vnd.spring-boot.ac
tuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.LoggersMvcEndpoint.get(java.lang.String)
2019-06-06 08:59:40.830  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/loggers/{name:.*}],methods=[POST],consumes=[application/vnd.spring-boot.a
ctuator.v1+json || application/json],produces=[application/vnd.spring-boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoin
t.mvc.LoggersMvcEndpoint.set(java.lang.String,java.util.Map<java.lang.String, java.lang.String>)
2019-06-06 08:59:40.831  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/loggers || /loggers.json],methods=[GET],produces=[application/vnd.spring-
boot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2019-06-06 08:59:40.832  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/info || /info.json],methods=[GET],produces=[application/vnd.spring-boot.a
ctuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.EndpointMvcAdapter.invoke()
2019-06-06 08:59:40.834  INFO 2820 --- [           main] o.s.b.a.e.mvc.EndpointHandlerMapping     : Mapped "{[/health || /health.json],methods=[GET],produces=[application/vnd.spring-bo
ot.actuator.v1+json || application/json]}" onto public java.lang.Object org.springframework.boot.actuate.endpoint.mvc.HealthMvcEndpoint.invoke(javax.servlet.http.HttpServletRequest,jav
a.security.Principal)
```
上面显示了一批端点的定义，这些端点并非我们自己在程序中创建的，而是由**spring-boot-starter-actuator**模块根据应用依赖和配置自动创建出来的监控和管理端点。通过这些端点，我们可以实时获取应用的各项监控指标，比如访问/health端点，我们可以获取如下信息：

```
{
    "status": "UP"
}
```

在没有引入其他依赖之前，该端点返回的内容较为简单，后续我们使用Spring Cloud的各个组件之后，它的返回会变得非丰富，这些内容将帮助我们制定更为人性化的监控策略。
## 2.2、actuator原生端点
actuator 的原生端点分为**3类**，如下所示：
1. **应用配置类**：获取应用程序中加载的应用配置、环境变量、自动化配置报告等于Spring Boot应用密切相关的配置类信息。
2. **度量指标类**：获取应用程序在运行过程中用于监控的度量指标，比如内存信息、线程池信息、HTTP请求统计等。
3. **操作控制类**：提供对应用的关闭等操作类功能。

下面让我们来举几个例子详细了解一下这三类原生端点，我们可以如何去扩展和配置它们。
### 2.2.1、应用配置类

由于Spring Boot为了改善传统Spring应用繁杂的配置内容，采用了包扫描和自动化配置的机制来加载原本集中在XML文件中的各项内容。虽然这样的做法让我们的代码变得非常简洁，但是整个应用的实例创建和依赖关系被分散在各个配置类注解上，这使得我们分析整个应用中的资源和实例关系变得非常困难。这类端点可以帮助我们轻松获取一系列关于Spring应用配置内容的详细报告，比如自动化配置的报告、Bean创建的报告、环境属性报告等。

1、**/autoconfig**:
该端点用于获取应用的自动化配置报告，其中，包括所有自动化配置的候选项。同时还列出了每个候选项是否满足自动化配置的各个先决条件。所以，该端点可以帮助我们方便找到一些自动化配置项为什么没有生效的具体原因。该报告的内容将自动化配置内容分为以下两部分：
- **positiveMaches**:返回的是条件匹配成功的自动化配置
- **negativeMaches**:返回的是条件匹配不成功的自动化配置

2、**/bean**:该端点用来获取上下文创建的所有Bean。


