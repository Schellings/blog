---
title: 为什么选Spring Cloud
date: 2019-01-17 19:56:24
tags: 微服务
---
# 1、Spring Cloud简介
Spring Cloud是一个基于Spring boot实现的微服务架构开发工具。它为微服务架构中涉及的配置管理、服务治理、断路由器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等操作提供了一种简单的开发方式。
> Spring Cloud—— 微服务架构集大成者，云计算最佳业务实践。

Spring Cloud中文网：https://springcloud.cc/

Spring Cloud包含了多个子项目（针对分布式系统中涉及的多个不同开源产品，还可能会新增），如下所述：
1. **Spring Cloud Config**:配置管理工具，支持使用Git存储配置内容，可以使用它实现应用配置的外部化存储，并支持客户端配置信息刷新、加密/解密配置内容等
2. **Spring Cloud Netfix**:核心组件，对多个Netflix OSS开源套件进行整合。如下所示：


- **Eureka**：服务治理组件，包含服务注册中心、服务注册与发现机制的实现。
- **Hystrix**：容错管理组件，实现断路由模式，帮助服务依赖中实现的延迟和为故障提供强大的容错能力。
- **Ribbon**：客户端负载均衡的服务调用组件。
- **Feign**：基于Ribbon和Hystrix的声明式服务调用组件。
- **Zuul**：网关组件，提供智能路由、访问过滤等功能。
- **Archaius**：外部化配置组件。
3. **Spring Cloud Bus**:事件、消息总线，用于传播集群中的状态变化或事件，以触发后续的处理，比如用来动态刷新配置等。
4. **Spring Cloud Cluster**：针对Zookeeper、Redis、Hazelcast、Consul的选举算法和通用状态的实现。
5. **Spring Cloud Cloudfoundry**：与Pivotal Cloudfoundry的整合支持。
6. **Spring Cloud Consul**：服务发现和配置管理工具。
7. **Spring Cloud Stream**：通过Redis、RabbitMQ或者Kafka实现的消费微服务，可通过简单的声明式模型来发送和接收消息。
8. **Spring Cloud AWS**：用于简化整合Amazon Web Service的组件。
9. **Spring Cloud Security**：安全工具包，提供在Zuul代理中对OAuth2客户端请求的中继器。
10. **Spring Cloud Sleuth**：Spring Cloud应用的分布式跟踪实现，可以完美整合Zipkin。
11. **Spring Cloud Zookeeper**：基于Zookeeper的服务发现和配置管理组件。
12. **Spring Cloud Starters**：Spring Cloud的基础组件，它是基于Spring boot风格项目的基础依赖模块。
13. **Spring Cloud CLI**：用于Groovy中快速构建Spring Cloud应用的Spring Boot CLI插件。
14. .....

常用子项目如下图所示：

![Spring Cloud常用子项目](https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=835037166,3207950096&fm=173&app=49&f=JPEG?w=640&h=532&s=D8AA3C72510A674D14611C460000E0B1)

其中，需要特别关注**Netfix**，如上所述，**Netfix**是整个Spring Cloud分布式架构实现的核心子项目。

# 2、Spring Cloud版本说明
在查看一些技术博客是常常发现，关于Spring Cloud的版本号认识不太清晰，如Angel.SR6、Brixton.SR5这似乎与Spring家族中以x.x.x版本号来指定版本的规则不同。之所以会出现这种情况，是因为Spring Cloud不是一个单独的项目，它内部包含着多个子项目，并且这些子项目可以独立的发布更新，各自有着自己的版本发布。

因此Spring Cloud作为一个多项目聚合项目的版本命名来源是伦敦的地铁站名，以字母排序。比如最早的Release版本为Angel，第二个Release版本为Brixton。当一个版本的update积累的比较多或者解决了一个严重bug时，会发布一个Service Release版本，简称SR。并且通过SRX的形式来标志第X个版本,其中X是一个递增的数字。

通过上面的解释，我们似乎认识到Angel.SR6表示**Angel的第6个发行版本**，Brixton.SR5指的是**Brixton的第5个发行版本**。


---

在本节我们大致了解了Spring Cloud的优点并与各个常用子项目先碰了个面，由于Spring Cloud是基于Spring Boot构建的，那么在下一节，那么在下一节我们将学习==Spring Boot快速入门==
