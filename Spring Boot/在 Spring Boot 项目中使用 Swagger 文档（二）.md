---
title: 在 Spring Boot 项目中使用 Swagger 文档（二）
date: 2017-07-16 19:56:24
tags: 
    - Spring Boot
    - Swagger
---
<meta name="referrer" content="no-referrer" />
## 高级配置

### 文档相关描述配置

（1）通过在控制器类上增加@Api 注解，可以给控制器增加描述和标签信息。

给 Controller 添加描述信息：

```java
@Api(tags = "用户相关接口", description = "提供用户相关的 Rest API")
public class UserController
```

（2）通过在接口方法上增加 @ApiOperation 注解来展开对接口的描述，当然这个注解还可以指定很多内容，我们在下面的相关注解说明章节中详细解释。

给接口添加描述信息：

```java
@ApiOperation("新增用户接口")
@PostMapping("/add")
public boolean addUser(@RequestBody User user) {
    return false;
}
```

（3）实体描述，我们可以通过 @ApiModel 和 @ApiModelProperty 注解来对我们 API 中所涉及到的对象做描述。

给实体类添加描述信息：

```java
@ApiModel("用户实体")
public class User {
    @ApiModelProperty("用户 id")
    private int id;
}
```

（4）文档信息配置，Swagger 还支持设置一些文档的版本号、联系人邮箱、网站、版权、开源协议等等信息。

但与上面几条不同的是这些信息不是通过注解配置，而是通过创建一个 ApiInfo 对象，并且使用 Docket.appInfo() 方法来设置。

我们在 SwaggerConfig.java 类中新增如下内容即可：

```java
@Bean
public Docket api() {
return new Docket(DocumentationType.SWAGGER_2)
.select()
            .apis(RequestHandlerSelectors.any())
            .paths(PathSelectors.any())
            .build()
            .apiInfo(apiInfo());
}
private ApiInfo apiInfo() {
return new ApiInfo(
            "Spring Boot 项目集成 Swagger 实例文档",
            "我的博客网站：https://itweknow.cn，欢迎大家访问。",
            "API V1.0",
            "Terms of service",
            new Contact("名字想好没", "https://itweknow.cn", "gancy.programmer@gmail.com"),
                "Apache", "http://www.apache.org/", Collections.emptyList());
}
```

经过上面的步骤，我们的文档将会变成下图的样子，现在看起来就清楚很多了。
![](https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/image002.png)

## 接口过滤

有些时候我们并不是希望所有的 Rest API 都呈现在文档上，这种情况下 Swagger2 提供给我们了两种方式配置，一种是基于 @ApiIgnore 注解，另一种是在 Docket 上增加筛选。

（1）@ApiIgnore 注解。

如果想在文档中屏蔽掉删除用户的接口（user/delete），那么只需要在删除用户的方法上加上 @ApiIgnore 即可。

```java
@ApiIgnore
public boolean delete(@PathVariable("id") int id)
```
（2）在 Docket 上增加筛选。Docket 类提供了 apis() 和 paths()两 个方法来帮助我们在不同级别上过滤接口：
```yaml
apis()：这种方式我们可以通过指定包名的方式，让 Swagger 只去某些包下面扫描。
paths()：这种方式可以通过筛选 API 的 url 来进行过滤。
```

在集成 Swagger2 的章节中我们这两个方法指定的都是扫描所有，没有指定任何过滤条件。如果我们在我们修改之前定义的 Docket 对象的 apis() 方法和 paths() 方法为下面的内容，那么接口文档将只会展示 /user/add 和 /user/find/{id} 两个接口。

```java
.apis(RequestHandlerSelectors.basePackage("cn.itweknow.sbswagger.controller"))
.paths(Predicates.or(PathSelectors.ant("/user/add"),
        PathSelectors.ant("/user/find/*")))
```

经过筛选过后的 Swagger 文档界面:

![](https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/image003.png)

## 自定义响应消息

Swagger 允许我们通过 Docket 的 globalResponseMessage() 方法全局覆盖 HTTP 方法的响应消息，但是首先我们得通过 Docket 的 useDefaultResponseMessages 方法告诉 Swagger 不使用默认的 HTTP 响应消息。

假设我们现在需要覆盖所有 GET 方法的 500 和 403 错误的响应消息，我们只需要在 SwaggerConfig.java 类中的 Docket Bean 下添加如下内容：

```java
.useDefaultResponseMessages(false)
.globalResponseMessage(RequestMethod.GET, newArrayList(
new ResponseMessageBuilder()
              .code(500)
              .message("服务器发生异常")
              .responseModel(new ModelRef("Error"))
              .build(),
       new ResponseMessageBuilder()
              .code(403)
              .message("资源不可用")
              .build()
));
```

添加如上面的代码后，如下图所示，您会发现在 SwaggerUI 页面展示的所有 GET 类型请求的 403 以及 500 错误的响应消息都变成了我们自定义的内容。

![](https://www.ibm.com/developerworks/cn/java/j-using-swagger-in-a-spring-boot-project/image004.png)

## 相关注解说明

### Controller 相关注解

@Api: 可设置对控制器的描述。

注解属性|	类型	|描述
---|---|---
tags|	String[]|	控制器标签。
description	|String|	控制器描述（该字段被申明为过期）。

### 接口相关注解

@ApiOperation: 可设置对接口的描述。

注解属性|	类型|	描述
---|---|---
value|	String|	接口说明。
notes|	String|	接口发布说明。
tags|	Stirng[]|	标签。
response|	Class<?>|	接口返回类型。
httpMethod|	String|	接口请求方式。

@ApiIgnore: Swagger 文档不会显示拥有该注解的接口。

@ApiImplicitParams: 用于描述接口的非对象参数集。

@ApiImplicitParam: 用于描述接口的非对象参数，一般与 @ApiImplicitParams 组合使用。

@ApiImplicitParam 主要属性:

注解属性|	描述
---|---
paramType|	查询参数类型，实际上就是参数放在那里。取值：path：以地址的形式提交数据，根据 id 查询用户的接口就是这种形式传参。query：Query string 的方式传参。header：以流的形式提交。form：以 Form 表单的形式提交。
dataType|	参数的数据类型。取值：Long、String
name|参数名字。
value|参数意义的描述。
required|是否必填。取值：true：必填参数。false：非必填参数。

### Model 相关注解

@ApiModel: 可设置接口相关实体的描述。
@ApiModelProperty: 可设置实体属性的相关描述。

@ApiModelProperty 主要属性：

注解属性|	类型|	描述
---|---|---
value|	String|	字段说明。
name	|String	|重写字段名称。
dataType|	Stirng|	重写字段类型。
required|	boolean	|是否必填。
example|	Stirng|	举例说明。
hidden|	boolean	|是否在文档中隐藏该字段。
allowEmptyValue|	boolean|	是否允许为空。
allowableValues	|String	|该字段允许的值，当我们 API 的某个参数为枚举类型时，使用这个属性就可以清楚地告诉 API 使用者该参数所能允许传入的值。