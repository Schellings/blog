---
title: Spring Boot初识
date: 2019-01-20 19:56:24
tags: 微服务
---
# 1、前言
在展开Spring Cloud的微服务架构部署之前，我们先通过本系列的内容了解一下用于构建微服务的基础架构——**Spring Boot**。对于Spring Boot已经有了深入了解的读者可以直接跳过本节，直接进入下一系列学习Spring Cloud各个组件的使用。

如果对于Spring Boot还不了解的话，建议读完本系列之后在继续学习后续关于Spring Cloud的内容。由于Spring Cloud的构建是基于Spring Boot实现，在后续的实例中我们将大量使用Spring Boot来构建微服务架构的基础设施以及一些实验中使用的微服务。为了能够辅助后续内容的介绍，确保读者有一定的Spring Boot基础，在这里先把Spring Boot做一个简单介绍，以保证读者有一定基础去理解后续介绍的Spring Cloud内容并顺利完成后续的一些示例实验。

在本系列中我们将介绍以下内容：

- 如何构建Spring Boot项目
- 如何实现Restful API接口
- 如何实现多环境的Spring Boot应用配置
- 深入理解Spring Boot配置的启动机制
- Spring Boot应用的监控与管理。

更多关于Spring Boot的使用细节，读者可以查看Spring Boot的官方文档或是其他资料进一步学习。

# 2、Spring Boot框架简介
对于很多Spring 框架的初学者来说，经常会因为复杂而繁琐的配置文件而对项目开发望而却步。而对于老鸟来说，每一次新建项目总会复制粘贴一些差不多的配置文件，这样的重复性工作我们应当尽可能避免。我们总会想尽办法来避免这种重复性的工作，比如通过maven等构建工具来创建针对不同场景的脚手架工程，在需要新建项目是通过这些脚手架来初始化我妈们自定义的标准工程，并根据需要进行简单的修改以达到简化原有配置过程得到效果。这样的做法虽然减少了工作量，但是这些配置依然大量散布于我们的工程中，大部分情况下我们并不会去修改这些内容，但为什么反复出现在我们的工程中呢？实在有些碍眼！

Spring Boot由此诞生了，Spring Boot的出现可以有效改善这一类问题，Spring Boot的宗旨并非要重写Spring或者替代Spring，而是希望通过设计大量的自动化配置方式来简化Spring原有配置，使得开发者可以**快速构建应用**。

除了解决配置问题之外，Spring Boot还通过一系列Starter POMs的定义，让我们整合各项功能的同时，不需要在maven的pom.xml中维护那错综复杂的依赖关系，而是通过类似模板化starter模块定义来引用，使得依赖管理工作变得更为简单。

在如今容器化技术大行其道的时代，Spring Boot除了可以很好的融入docker之外，其自身就支持嵌入式的tomcat、jetty等web容器。所以，通过Spring Boot构建的应用不再需要安装tomcat，再将应用打包成war包，再部署到tomcat中这样复杂的构建和部署动作，只需要将Spring Boot应用打包成jar包，并通过**java -jar**命令直接运行就能启动一个标准化的web应用，这使得Spring Boot应用变得非常轻便。

Spring Boot对于构建、部署等做了大量优化，自然也少不了对开发环节的优化。整个Spring Boot的生态系统中都使用到了Groovy，我们完全可以使用Gradle和Groovy来开发Spring Boot应用。

说了这么多Spring Boot带来的颠覆性框架特点，我们来通过快速入门部分对Spring Boot有更直观的感受。

# 3、快速入门
在本节中，我们通过一个简单的Spring Boot项目并实现一个简单的Restful API来对Spring Boot更直观的感受一下。
## 3.1 环境要求
- jdk 1.8.0_73及以上
- Spring Framework 4.2.7及以上版本
- Maven 3.2及以上版本

（以下代码在环境jdk 1.8.0_73、Spring Boot 1.5.21 调试通过）
## 3.2、Maven项目构建
1. 通过Spring官方项目构建工具来产生基础项目
2. 访问 https://start.spring.io/ 该页面提供以Maven或Gradle构建Spring Boot项目
3. 选择Maven、Java
4. 选择Spring Boot版本为1.5.21
5. 填写Project Metadata信息
6. 由于我们是构建一个基础的java web项目，需要在Dependencies栏搜索"web"并添加web依赖。
7. 点击generate project按钮下载初始项目
8. 解压，并使用idea导入该项目
9. 等待项目构建，解决相关pom依赖
## 3.3、工程结构解析
初始项目工程截图如下所示：

![初始项目demo-01工程截图](http://chatbot.xielin.top/img/demo01-1.jpg)

1. src/main/java/项目名Application.java是Spring Boot项目启动文件。
2. src/main/resource下保存的是项目资源文件，该目录用于保存项目配置文件，由于我们导入了web依赖，因此会创建static、templates文件夹，前者用于保存css、js、图片等，后者用于保存web页面的模板文件。由于本次demo仅仅用于创建一个Restful API，因此这两个文件夹都不会被用到。
3. src/test 单元测试文件夹，生成的Demo01ApplicationTests文件通过JUnit4实现，可以直接用于测试Spring Boot应用。在后文中，我们会演示如何在该类中测试Restful API。
## 3.4、pom文件解析
打开当前工程下的pom.xml文件，看看生成的项目都引入了哪些依赖来构建Spring Boot工程，内容大致如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<packaging>jar</packaging>
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.5.21.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
	<groupId>com.gnnu</groupId>
	<artifactId>demo-01</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>demo-01</name>
	<description>Demo project for Spring Boot</description>

	<properties>
		<java.version>1.8</java.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
</project>

```

我们注意到groupId和artifactId就是在生成项目的Spring网页上输入的内容，我们注意到打包形式为：**<packaging>jar</packaging>**，（如果没有该项配置，则默认打包方式就是jar）正如我们前面所介绍的，Spring Boot默认将web应用打包为jar的形式，而非war的形式，因为默认的web模块依赖会包含嵌入式tomcat，这样使得我们的应用jar自身就具备了提供web服务的能力，后续我们会演示如何启动它。

父项目parent配置项中指定的spring-boot-starter-parent版本为1.5.21版本，该父项目中定义了Spring Boot版本的基础依赖以及一些默认配置内容1，比如，配置文件application.properties的位置等。

在项目依赖dependencies配置中，包含以下两项：
1. **spring-boot-starter-web**：全栈web开发模块，包含嵌入式tomcat、Spring MVC等。
2. **spring-boot-starter-test**：通用测试模块，包含JUnit、Hamcrest、Mockito。

这里所引用的web和test模块，在Spring Boot生态中被称为**Starters POMs**。Starters POMs是一系列轻便的依赖包，是一套一站式的Spring相关技术的解决方案。开发者在使用和整合模块是，不必再去搜寻样例代码找那个的依赖配置来复制使用，只需要引入对应的模块包即可。比如，开发应用时，就只需要引入spring-boot-starter-web；希望应用具备访问数据库能力是，那就再引入spring-boot-starter-jdbc,或是更好用的spring-boot-starter-data-jpa。在使用Spring Boot构建应用时，各项功能模块的整合不再像传统Spring应用的开发方式那样。需要在pom.xml文件中做大量的依赖配置，而是通过使用Starters POMs定义的依赖包，使功能模块整合变得非常轻巧，易于理解和使用。

Spring Boot的Starters POMs采用
**Spring-boot-starter-***
的命名方式，*代表一个特别的应用功能模块，比如这里的web、test。Spring Boot工程本身的结构非常简单，大量的学习要点还是将来在对这些Starters POMs的使用之上。在本系列中主要讲述Spring Cloud的微服务组件内容，因此对各个Spring Boot的模块内容不做详细的讲解。对于初学者来说，我们不必一次性将所有模块的详细用法掌握牢固，只需要了解每个模块能做什么即可。

下面对一些常见的Starters POMs进行简要说明：
![常用Starters POMs](http://chatbot.xielin.top/img/demo01-3.png)

最后关于项目的build部分，前面我们说过可以在src/main/java下找到“项目名Application.java”文件运行整个Spring Boot项目，但是更为简便的方式是在console中使用命令**mvn spring-boot:run**命令快速启动Spring Boot应用。

## 3.5、实现Restful API
自此关于Spring Boot的项目已经有了初步认识，让我们来实现一个Restful API接口。

在Spring Boot中创建一个Restful API的实现代码同Spring MVC应用一样，只是不需要像Spring MVC那样先做诸多配置，而是像下面这样直接开始编写Controller内容：

- 新建package，命名为com.gnnu.web，可根据实际情况自定义包名。
- 新建HelloController类，内容如下：

```
@RestController
public class HelloController {
    @RequestMapping(value = "/hello",method = RequestMethod.GET)
    public String index(){
        return "Hello World";
    }
}

```

## 3.6、启动Spring Boot应用
前面我们或多或少说明过如何启动Spring Boot应用，启动方式多种多样，在此处做个汇总，主要包含以下几种方式:

1. 作为一个java应用，必然可以通过执行main函数来启动，即进入文件“src/main/java/项目名Application.java”右击run即可。
2. 由于项目是通过maven来构建的，并且仔细查看pom.xml文件，在plugins配置项中引入了
**spring-boot-maven-plugin**插件，该插件支持Spring
Boot应用在项目根路径下使用命令**mvn spring-boot:run**来运行项目。

常用的命令如下图所示:

![spring-boot:xxx命令](http://chatbot.xielin.top/img/demo01-7.jpg)

3.前面说过Spring  Boot项目打包为jar包，那么服务器端我们可以先通过
**mvn install** 命令打包jar包，进入项目根目录/target/下使用命令java -jar xxx.jar将Spring Boot项目跑起来。

## 3.7、运行踩坑

启动运行Spring Boot项目，在浏览器访问链接 http://localost:8080/hello 如果运行正常，那么就能在页面上看到Hello World字样。但是，事情还没有结束！我们现在看到了一个熟悉的页面：

![404访问出错](http://chatbot.xielin.top/img/demo01-4.jpg)

哦豁，那为什么会出现这个问题呢？

让我们看到一段启动日志信息:

```
2019-06-03 18:20:31.916  INFO 8276 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2019-06-03 18:20:31.917  INFO 8276 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2019-06-03 18:20:31.933  INFO 8276 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
2019-06-03 18:20:31.934  INFO 8276 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
```
首先这段启动日志列出了该Spring Boot项目映射的URL路由信息，似乎我们自己编写的
**/hello**路径并没有出现，我们现在回想一下，我们似乎并没有告诉Spring Boot项目从何处扫描加载Controller。那在何处设置扫描路径呢？

前面说过，Spring Boot对项目配置进行了大量优化配置，我们可以通过注解的方式修改扫描路径。

进入到项目名Application.java文件，修改注解默认配置，我这里是Demo01Application.java，代码如下：

```
package com.gnnu.demo01;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication(scanBasePackages = {
		"com.gnnu"
})
public class Demo01Application {

	public static void main(String[] args) {
		SpringApplication.run(Demo01Application.class, args);
	}

}

```
默认内容为：

```
package com.gnnu.demo01;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Demo01Application {

	public static void main(String[] args) {
		SpringApplication.run(Demo01Application.class, args);
	}

}
```
比较发现，在
**@SpringBootApplication**注解中设置了一个参数
**scanBasePackages**。该参数的默认设置为
**项目启动文件即项目名Application.java文件所在包下**Spring Boot会扫描这个包下（包含子包）的被
**@RestController**注解的类作为一个Controller，Controller下每一个被
**@RequestMapping**
注解的方法作为一个路由映射，用于处理一个URL请求。
我们编写的方法并不在默认配置的包下。

经过上诉修改之后，重新运行项目，Spring Boot能够正常扫描到该Controller类并我们能够正常访问链接 http://localost:8080/hello 

![正确访问截图](http://chatbot.xielin.top/img/demo01-6.jpg)

同时启动日志信息也表明路由信息正确加载：

```
2019-06-03 18:46:28.441  INFO 13940 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/hello],methods=[GET]}" onto public java.lang.String com.gnnu.demo01.HelloController.index()
2019-06-03 18:46:28.444  INFO 13940 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error]}" onto public org.springframework.http.ResponseEntity<java.util.Map<java.lang.String, java.lang.Object>> org.springframework.boot.autoconfigure.web.BasicErrorController.error(javax.servlet.http.HttpServletRequest)
2019-06-03 18:46:28.444  INFO 13940 --- [           main] s.w.s.m.m.a.RequestMappingHandlerMapping : Mapped "{[/error],produces=[text/html]}" onto public org.springframework.web.servlet.ModelAndView org.springframework.boot.autoconfigure.web.BasicErrorController.errorHtml(javax.servlet.http.HttpServletRequest,javax.servlet.http.HttpServletResponse)
2019-06-03 18:46:28.459  INFO 13940 --- [           main] o.s.w.s.handler.SimpleUrlHandlerMapping  : Mapped URL path [/webjars/**] onto handler of type [class org.springframework.web.servlet.resource.ResourceHttpRequestHandler]
```
## 3.8、编写单元测试
功能实现后，我们要养成随手写配套单元测试的习惯，这在微服务架构中尤为重要。我们在实施微服务架构时候，已经实现了前后端分离的项目与架构部署。那么在实现前后端服务的时候，单元测试是在开发环境中用于验证代码正确性的非常好的手段，并且这些单元测试将会很好支持我们未来可以进行的重构。

在Spring Boot中实现单元测试非常方便，前面我们说过在项目pom.xml文件中导入了 **spring-boot-starter-test**组件，因此具备单元测试环境。
下面我们打开src/test下的测试入口，新建一个HelloControllerTests类，内容如下：

```
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.SpringBootConfiguration;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.hamcrest.core.IsEqual.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

/**
 * @Classname HelloControllerTests
 * @Description TODO
 * @Date 2019/6/3 19:28
 * @Created by XL
 */
@RunWith(SpringJUnit4ClassRunner.class)
@WebAppConfiguration
public class HelloControllerTests {
    private MockMvc mvc;

    @Before
    public void before()throws Exception{
        mvc= MockMvcBuilders.standaloneSetup(new HelloController()).build();
    }

    @Test
    public void hello() throws Exception{
        mvc.perform(MockMvcRequestBuilders.get("/hello").accept(MediaType.APPLICATION_JSON))
        .andExpect(
                status().isOk()
        ).andExpect(
                content().string(equalTo("Hello World")
                )
        );
    }
}

```
代码解析如下：
1. **@RunWith(SpringJUnit4ClassRunner.class)**：引入Spring对JUnit4的支持。
2. **@WebAppConfiguration**：开启web应用配置，用于模拟ServletContext。
3. MockMvc对象：用于模拟调用Controller的接口发起请求，在@Test定义的hello测试用例中，perform函数执行一次请求调用，accept用于执行接收数据类型，andExpect用于判断接口返回的期望值。
4. **@Before**：JUnit中定义测试用例@Test内容执行前预加载的内容，这里用来初始化对HelloController的模拟。

运行HelloControllerTests类将执行里面所有的测试用例，执行结束发现测试用例绿灯pass，修改测试用例内容：

```
content().string(equalTo("Hello World")
// 修改为
content().string(equalTo("Hello World2")
```
重新执行会发现测试用例红灯failed


---

自此Spring Boot快速入门已结束，我们对Spring Boot项目有了一个大致的认识。那么在下一节我们将学习==深入认识Spring Boot项目配置==