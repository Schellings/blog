---
title: Spring Boot深入配置
date: 2019-01-22 16:56:24
tags: 微服务
---
# 1、前言
在上一节，我们对Spring Boot有了一个大致的认识，通过实现的一个简单的Restful API应用，体验了Spring Boot的诸多优点。虽然在实现Controller中实现的代码大体一致，但是在配置方面，除了maven中对Starters POMs的依赖配置，基本上没有引入其他配置，这大大减小了我们在项目初始模板创建时的难度。

但是大量的自动化配置项在一些特殊的应用场景并不适用，例如，本地8080端口已经被一个Spring Boot应用给占用了，在不释放该端口占用的情况下，默认的端口部署配置必将导致端口冲突使得下一个Spring Boot应用启动失败。

在后续的Spring Cloud学习中，其实我们大量的工作都会针对各个组件的配置文件进行自定义修改。所以我们需要深入了解Spring Boot项目配置文件的知识。比如**配置方式、如何实现多环境、配置信息加载顺序等**

# 2、配置文件
在快速入门的例子中，我们介绍Spring Boot的工程结构时，提到过src/main/resources目录是Spring Boot的配置目录，所以当要为应用创建个性化配置时，应该在该目录下进行。

## 2.1、传统properties配置文件
Spring Boot的默认配置文件位置为src/main/resources/application.properties。关于Spring Boot应用的配置内容都可以集中到该文件中，根据我们引入的不同Starter模块，可以在这里定义tomcat容器启动端口、数据库连接信息、日志级别等各类配置信息。比如，我们需要自定义web模块的服务端口号即tomcat容器启动端口，可以在application.properties文件中添加server.port=8888来指定启动端口为8888，也可以通过spring.application.name=hello来指定应用名（该名字在后续Spring Cloud中会被注册为服务名）

即在application.properties文件中的内容如下:

```
server.port=8888
spring.application.name=hello
```
重启项目，发现项目启动端口已被修改为8888

Spring Boot的配置文件除了可以使用传统的properties文件外，还支持现在被广泛使用的**YAML**文件

## 2.2、YAML初识
YAML 是一种简洁的非标记语言。YAML以数据为中心，使用空白，缩进，分行组织数据，从而使得表示更加简洁易读。

让我们先看两段配置文件信息：
1. 传统properties格式配置文件

```
environments.dev.url=http://dev.bar.com
environments.dev.name=Developer Setup
environments.prod.url=http://foo.bar.com
environments.prod.name=My Cool App
my.servers[0]=dev.bar.com
my.servers[1]=foo.bar.com
```

2. YAML格式配置文件

```
environments:
    dev:
        url: http://dev.bar.com
        name: Developer Setup
    prod:
        url: http://foo.bar.com
        name: My Cool App
my:
    servers:
        - dev.bar.com
        - foo.bar.com
```

上述两个配置文件的所配置的信息是等价的，但是显然我们发现，YAML格式的配置文件信息更为简洁，可读性更好，在处理多层级配置信息是，YAML格式文件的优势被大大凸显出来。

语法方面可以直观的看到，YAML使用冒号加缩进的方式代表层级（属性）关系，使用短横杠(-)代表数组元素。

快速开始的例子中的src/main/resources目录下新建一个文件application.yaml，内容如下：


```
server:
  port: 8888
spring:
  application:
    name: hello
```
重启项目，发现与传统properties配置文件的效果一致，项目启动端口已被修改为8888

推荐大家多多使用YAML作为Spring Boot项目配置文件格式，会在项目配置上大大减轻工作量。关于YAML语法方面在这里就不多展开介绍，大家可以查看其它相关资料深入学习。

**注：**

**1、当properties文件与YAML配置文件同时存在时，会默认使用properties文件**

**2、虽然YAML有很多优点，但是YAML同样有一些不足，例如它无法通过@PropertySource注解来加载配置信息**

**3、YAML加载到内存中保存的时候是有序的，所以当配置文件中信息需要具备顺序含义时，YAML的配置方式比起properties配置文件更有优势**

## 2.3、自定义参数
除了可以在Spring Boot的配置文件中设置各个Starter模块中预定义的属性外，也可以在配置文件中定义一些我们需要的自定义属性。比如，在application.yaml中添加以下内容：

```
book:
  name: 《Spring Boot blog》
  author: XL
```
在应用中可以通过 **@Value** 注解来引用该配置信息，比如：

```
@Component
public class Book {
    @Value("${book.name}")
    private String name;
    @Value("${book.author}")
    private String author;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }
}
```
@Value注解加载属性值的时候支持两种表达式来进行使用，如下所示：
1. 一种是上面使用的PlaceHolder方式，格式为${...}，大括号内为PlaceHolder。
2. 另一种是使用SpEL表达式（Spring Expression Language），格式为#{...}，大括号内为SpEL表达式。
## 2.4、参数引用
在application.yaml配置文件中的各个参数之间还可以直接通过PlaceHolder的方式来进行引用，例如：

```
book:
  name: 《Spring Boot blog》
  author: XL
  desc: ${book.author} is Writing ${book.name}
```
## 2.5、random随机数
在一些特殊情况下，我们希望有些参数每次被加载是不是一个固定值，比如密钥、服务端口等。在Spring Boot的属性配置文件中，可以通过使用${random}配置来产生随机的int值、long值或者string字符串，这样我们就可以容易的通过配置随机生成属性，而不是在程序中通过编码来实现这些逻辑。

示例如下：

```
random:
  number:
    intValue: ${random.int}
    strValue: ${random.value}
    longValue: ${random.long}
    # 10以内的随机数
    test01: ${random.int(10)}
    # 10-20的随机数
    test02: ${random.int[10,20]}
```
该配置可用于设置应用端口等场景，以避免在本地调试时出现端口冲突的麻烦。

## 2.6、命令行参数

在上一小节快速入门中，我们介绍了Spring Boot项目可以通过命令**java -jar xxx.jar**来启动项目，该命令除了启动应用之外，还可以在命令行中指定应用配置参数，比如**java -jar xxx.jar --server.port=8888**，直接以命令行的方式来设置server.port属性，并将应用启动端口设置为8888。

在使用命令行方式启动Spring Boot应用时，**--xxx**(英文状态下两个连续的-)就是对application.yaml中的属性值进行赋值的标识。所以，**java -jar xxx.jar --server.port=8888**命令等价于在application.yaml中设置属性

```
server:
  port: 8888
```
并使用命令**java -jar xxx.jar**启动项目。

通过命令行来修改属性值是Spring Boot非常重要的一个特征。通过此特征理论上application.yaml配置文件中的所有属性都可以被修改，启动端口也好、数据库连接信息也好，都可以在Spring Boot应用启动时被修改。但是如果每个参数都需要通过命令行来指定，这显然不是一个好的方案，所有让我们来看看如何在Spring Boot中实现多环境的配置。

## 2.7、多环境配置
我们在开发应用时，通常同一套程序会被应用和安装到好几个不同的环境中。比如开发、测试、生产等。其中每个环境的数据库信息、服务器端口等配置都不同。如果在为不同的环境打包时都要频繁的修改配置文件的话，那必将是一个非常繁琐而且容易出错的事。

对于多环境的配置，各种项目构建工具或是框架的基本思路都是一致的通过配置多份不同环境的配置文件，再通过打包命令指定需要打包的内容之后进行区分打包，Spring Boot也不例外，或者说更为简单。

在Spring Boot中，多环境配置的文件名需要满足application-[profile].yaml的格式，其中{profile}对应你的环境识别，如下所示：
1. **application-dev.yaml**：开发环境
2. **application-test.yaml**：测试环境
3. **application-prod.yaml**：生产环境

具体哪个配置文件会被加载，需要在application.yaml中通过设置spring.profiles.active属性，其对应的值就是各种配置文件名中的{profile}值，如spring.profiles.active=test就会加载application-test.yaml配置文件的内容。

同时结合2.6节介绍的命令行参数特性，使用命令**java -jar xxx.jar spring.profile.active=dev**可激活使用开发环境。

# 3、配置加载顺序
在上面的例子中，我们将Spring Boot应用需要的配置内容都放到了项目工程中，已经能够通过**spring.profile.active**参数再结合application-{profile}.yaml来实现多环境支持。但是，当团队逐渐扩大时，分工越发细致之后，往往不需要让开发人员知道测试或生产环境的细节，而是希望每个环境各自的负责人来集中维护这些信息，那么如果还是以这种形式来存储配置信息，对于不同环境的修改就不得不或许项目工程来修改这些配置信息，当应用过多时就变得非常不方便。

同时，配置内容对于开发人员都可见，这本身就是一种安全隐患。对此，出现了很多将配置内容外部化的框架和工具，后续将要介绍的Spring Cloud Config就是其中之一。为了后续能更好的理解Spring CloudConfig的加载机制，我们需要对Spring Boot的数据文件加载机制有一定的了解。

为了能够更合理的重写各属性的值，Spring Boot使用下面这种较为特别的属性加载顺序：

![配置信息加载顺序](http://chatbot.xielin.top/img/demo01-8.jpg)

加载优先级由上面的顺序由高到低，前面的数值越小，优先级越大。

可以看到，其中第7项和第9项都是从应用jar包之外读取配置文件，所以实现外部化配置的原理就是在此处切入，为其指定外部配置文件的加载位置来取代jar包之内的配置信息。通过这样的实现，我们的工程在配置中就变得非常干净，只需要在本地放置开发需要的配置即可，而不需要关系其他环境的配置，由其对应负责人去维护即可。


---
本节对Spring Boot项目配置有了大致了解，接下来，我么将继续学习==Spring Boot中的项目监控管理==。