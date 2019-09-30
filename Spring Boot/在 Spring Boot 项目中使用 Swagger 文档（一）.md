---
title: 在 Spring Boot 项目中使用 Swagger 文档（一）
date: 2017-07-14 19:56:24
tags: 
    - Spring Boot
    - Swagger
---

对于 Rest API 来说很重要的一部分内容就是文档，Swagger 为我们提供了一套通过代码和注解自动生成文档的方法，这一点对于保证 API 文档的及时性将有很大的帮助。

本文将使用 Swagger 2 规范的 Springfox 实现来了解如何在 Spring Boot 项目中使用 Swagger。

主要包含了如何使用 Swagger 自动生成文档、使用 Swagger 文档以及 Swagger 相关的一些高级配置和注解。

## Swagger 简介

Swagger 是一套基于 OpenAPI 规范构建的开源工具，可以帮助我们设计、构建、记录以及使用 Rest API。

Swagger 主要包含了以下三个部分：

- Swagger Editor：基于浏览器的编辑器，我们可以使用它编写我们 OpenAPI 规范。
- Swagger UI：它会将我们编写的 OpenAPI 规范呈现为交互式的 API 文档，后文我将使用浏览器来查看并且操作我们的 Rest API。
- Swagger Codegen：它可以通过为 OpenAPI（以前称为 Swagger）规范定义的任何 API 生成服务器存根和客户端 SDK 来简化构建过程。

## 为什么要使用 Swagger

当下很多公司都采取前后端分离的开发模式，前端和后端的工作由不同的工程师完成。

在这种开发模式下，维持一份及时更新且完整的 Rest API 文档将会极大的提高我们的工作效率。

传统意义上的文档都是后端开发人员手动编写的，相信大家也都知道这种方式很难保证文档的及时性，这种文档久而久之也就会失去其参考意义，反而还会加大我们的沟通成本。

而 Swagger 给我们提供了一个全新的维护 API 文档的方式，下面我们就来了解一下它的优点：

- 代码变，文档变。只需要少量的注解，Swagger 就可以根据代码自动生成 API 文档，很好的保证了文档的时效性。
- 跨语言性，支持 40 多种语言。
- Swagger UI 呈现出来的是一份可交互式的 API 文档，我们可以直接在文档页面尝试 API 的调用，省去了准备复杂的调用参数的过程。
- 还可以将文档规范导入相关的工具（例如 SoapUI）, 这些工具将会为我们自动地创建自动化测试。

以上这些优点足以说明我们为什么要使用 Swagger 了，您是否已经对 Swagger 产生了浓厚的兴趣了呢？下面我们就将一步一步地在 Spring Boot 项目中集成和使用 Swagger，让我们从准备一个 Spring Boot 的 Web 项目开始吧。

## 基础环境

UserController.java 代码：

```java
@RestController
@RequestMapping("/user")
public class UserController {
    @PostMapping("/add")
    public boolean addUser(@RequestBody User user) {
        return false;
    }
    @GetMapping("/find/{id}")
    public User findById(@PathVariable("id") int id) {
        return new User();
    }
    @PutMapping("/update")
    public boolean update(@RequestBody User user) {
        return true;
    }
    @DeleteMapping("/delete/{id}")
    public boolean delete(@PathVariable("id") int id) {
        return true;
    }
}
```

## 集成 Swagger2

经过上面的步骤，我们已经拥有了五个接口，分别是:

- /user/add：新增用户。
- /user/find/{id}：根据 id 查询用户。
- /user/update：更新用户。
- /user/delete/{id}：根据 id 删除用户。
- /test/test：测试接口。

下面我们将通过集成 Swagger2，然后为这 5 个 Rest API 自动生成接口文档。

### 添加依赖

```yaml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.9.2</version>
</dependency>
```

### Java 配置
Springfox 提供了一个 Docket 对象，让我们可以灵活的配置 Swagger 的各项属性。

下面我们新建一个 cn.itweknow.sbswagger.conf.SwaggerConfig.java 类，并增加如下内容:

```java
@Configuration
@EnableSwagger2
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .select()
                .apis(RequestHandlerSelectors.any())
                .paths(PathSelectors.any())
                .build();
    }
}
```

注意: @Configuration 是告诉 Spring Boot 需要加载这个配置类，@EnableSwagger2 是启用 Swagger2，如果没加的话自然而然也就看不到后面的验证效果了。

### 验证

至此，我们已经成功的在 Spring Boot 项目中集成了 Swagger2，启动项目后，我们可以通过在浏览器中访问 http://localhost:8080/ v2/api-docs 来验证，您会发现返回的结果是一段 JSON 串，可读性非常差。

幸运的是 Swagger2 为我们提供了可视化的交互界面 SwaggerUI，下面我们就一起来试试吧。

## 集成 Swagger UI
### 添加依赖

```yaml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.9.2</version>
</dependency>
```
### 访问验证
其实就只需要添加一下依赖就可以了，我们重新启动一下项目，然后在浏览器中访问 http://localhost:8080/swagger-ui.html 

就可以看到如下的效果了:

![](https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/image001.png)

可以看到虽然可读性好了一些，但对接口的表述还不是那么的清楚，接下来我们就通过一些高级配置，让这份文档变的更加的易读。


