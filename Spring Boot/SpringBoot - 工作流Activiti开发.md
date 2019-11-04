---
title: SpringBoot - 工作流Activiti开发
date: 2019-02-29 16:56:24
tags: Spring Boot
---

## 环境安装

### IDEA安装actiBPM

通过File -> Settings -> Plugins 找到actiBPM插件进行安装

![](https://yqfile.alicdn.com/bb42a978a7f54ac07f29e42d2fc0d0ae42e227f2.jpeg)

### 核心API介绍

![](https://yqfile.alicdn.com/f215baa9207503132d919366fe4d0425891a2d64.png)

- ProcessEngineConfiguration:流程引擎配置。
- ProcessEngine:流程引擎

核心七大接口:

- RepositoryService：提供一系列管理流程部署和流程定义的API。
- RuntimeService：在流程运行时对流程实例进行管理与控制。
- TaskService：对流程任务进行管理，例如任务提醒、任务完成和创建任务等。
- IdentityService：提供对流程角色数据进行管理的API，这些角色数据包括用户组、用户及它们之间的关系。
- ManagementService：提供对流程引擎进行管理和维护的服务，提供对activiti数据库的直接访问【一般不用】
- HistoryService：对流程的历史数据进行操作，包括查询、删除这些历史数据。FormService：表单服务

### 添加依赖

```yaml
<dependency>
   <groupId>org.activiti</groupId>
   <artifactId>activiti-spring-boot-starter-basic</artifactId>
   <version>6.0.0</version>
</dependency>
```

### yml配置

```yaml
spring:
  activiti:
    check-process-definitions: true #自动检查、部署流程定义文件
    database-schema-update: true #自动更新数据库结构
    history-level: full #保存历史数据级别设置为full最高级别，便于历史数据的追溯
    # process-definition-location-prefix: classpath:/processes/ #流程定义文件存放目录
    #process-definition-location-suffixes: #流程文件格式
    #  - **.bpmn20.xml
    #  - **.bpmn
```

spring boot环境下不再以activiti.cfg.xml文件的形式配置，activiti使用starter配置后属于spring下，所以在yml里配置。

- check-process-definitions【检查Activiti数据表是否存在及版本号是否匹配】默认为true，自动创建好表之后设为false。设为false会取消自动部署功能。
- database-schema-update【在流程引擎启动和关闭时处理数据库模式】如下四个值:

        false （默认值）：在创建流程引擎时检查库模式的版本，如果版本不匹配则抛出异常。
        true：在创建流程引擎时，执行检查并在必要时对数据库中所有的表进行更新，如果表不存在，则自动创建。
        create-drop：在创建流程引擎时，会创建数据库的表，并在关闭流程引擎时删除数据库的表。
        drop-create：Activiti启动时，执行数据库表的删除操作，在Activiti关闭时，会执行数据库表的创建操作。
- history-level 【历史数据保存级别】

        none：不保存任何的历史数据，因此，在流程执行过程中，这是最高效的。
        activity：级别高于none，保存流程实例与流程行为，其他数据不保存。
        audit：除activity级别会保存的数据外，还会保存全部的流程任务及其属性。audit为history的默认值。
        full：保存历史数据的最高级别，除了会保存audit级别的数据外，还会保存其他全部流程相关的细节数据，包括一些流程参数等。

### 添加processes目录

SpringBoot集成activiti默认会从classpath下的processes目录下读取流程定义文件，所以需要在src/main/resources目录下添加processes目录，并在目录中创建流程文件。

如果没有processes目录，则需要修改配置spring.activiti.process-definition-location-prefix，指定流程文件存放目录。

![](https://yqfile.alicdn.com/025642a5d1c35cfbc57d57e4c1afcc96eec374eb.jpeg)

### 其他相关配置

现在系统的启动类排除org.activiti.spring.boot.SecurityAutoConfiguration即可。

```java
@SpringBootApplication(exclude = SecurityAutoConfiguration.class,scanBasePackages="com.poly")
public class AvtivityApplication {
    public static void main(String[] args) {
        SpringApplication.run(AvtivityApplication.class, args);
    }
}
```

官方文档给出的启动类配置:

```java
@Configuration
@ComponentScan
@EnableAutoConfiguration
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

### 配置bpmn

#### 通过插件配置

#### 官方文档

测试bpmn文件 命名为one-task-process.bpmn20.xml 放入processes目录
```yaml
<?xml version="1.0" encoding="UTF-8"?>
<definitions
        xmlns="http://www.omg.org/spec/BPMN/20100524/MODEL"
        xmlns:activiti="http://activiti.org/bpmn"
        targetNamespace="Examples">

    <process id="oneTaskProcess" name="The One Task Process">
        <startEvent id="theStart" />
        <sequenceFlow id="flow1" sourceRef="theStart" targetRef="theTask" />
        <userTask id="theTask" name="my task" />
        <sequenceFlow id="flow2" sourceRef="theTask" targetRef="theEnd" />
        <endEvent id="theEnd" />
    </process>

</definitions>
```
### 配置数据源

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/activiti_test?useUnicode=true&characterEncoding=UTF-8&serverTimezone=UTC
    username: root
    password: 12345678
```

> 执行完上述流程，启动项目后，生成如下数据库表：

![](https://yqfile.alicdn.com/da431769b049f6094382689dd5bf229e40cc7666.jpeg)

    act_ge_ 通用数据表，ge是general的缩写
    act_hi_ 历史数据表，hi是history的缩写，对应HistoryService接口
    act_id_ 身份数据表，id是identity的缩写，对应IdentityService接口
    act_re_ 流程存储表，re是repository的缩写，对应RepositoryService接口，存储流程部署和流程定义等静态数据
    act_ru_ 运行时数据表，ru是runtime的缩写，对应RuntimeService接口和TaskService接口，存储流程实例和用户任务等动态数据


    资源库流程规则表
    1) act_re_deployment 部署信息表
    2) act_re_model  流程设计模型部署表
    3) act_re_procdef  流程定义数据表
    
    运行时数据库表
    
    1) act_ru_execution运行时流程执行实例表
    2) act_ru_identitylink运行时流程人员表，主要存储任务节点与参与者的相关信息
    3) act_ru_task运行时任务节点表
    4) act_ru_variable运行时流程变量数据表
    
    历史数据库表
    
    1) act_hi_actinst 历史节点表
    2) act_hi_attachment历史附件表
    3) act_hi_comment历史意见表
    4) act_hi_identitylink历史流程人员表
    5) act_hi_detail历史详情表，提供历史变量的查询
    6) act_hi_procinst历史流程实例表
    7) act_hi_taskinst历史任务实例表
    8) act_hi_varinst历史变量表
    
    组织机构表
    
    1) act_id_group用户组信息表
    2) act_id_info用户扩展信息表
    3) act_id_membership用户与用户组对应信息表
    4) act_id_user用户信息表