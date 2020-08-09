---
title: Spring Boot 一些常用的注解介绍（二）
date: 2018-01-28 19:56:24
tags: Spring Boot
---
<meta name="referrer" content="no-referrer" />
## 一、简言
```yaml
在spring boot的开发中常常会用到注解 :

@RequestParam、@PathVariable、@RequestBody、@PageableDefault、@RequestMapping、@GetMapping 和 @PostMapping

定义提供给前端的接口函数。下面将会对这些注解进行说明。
```

## 二、注解说明
### @PathVariable
绑定URI模板变量值，用来获得url中的动态参数值。使用示例参考如下：

提供给前端的URL：
```yaml
http://localhost:8080/springmvc/hello/101?param1=10&param2=20 
```
后端接口代码如下：
```java
@RequestMapping("/hello/{id}")
    public String getDetails(@PathVariable(value="id") String id,
    @RequestParam(value="param1", required=true) String param1,
    @RequestParam(value="param2", required=false) String param2){
    .......
}
```
在后端的接口定义中，定义了三个参数id，param1，param2。

### @RequestParam
从request里面拿取值。使用示例参考如下：

提供给前端的URL：
```yaml
http://localhost:8080/springmvc/hello/101?param1=10&param2=20
```
后端提供的接口代码如下：
```java
public String getDetails(
    @RequestParam(value="param1", required=true) String param1,
    @RequestParam(value="param2", required=false) String param2){
    ...
}
```
@RequestParam支持四种参数:

- defaultValue如果本次请求没有携带这个参数，或者参数为空，那么就启用默认值。
- name绑定本次参数的名称，要跟URL上面的一样
- required这个参数表明，参数值是不是必须的，true表示必须不能为空，false表示不是必须可以为空
- value 跟name一样的作用，是name属性的一个别名。

### @RequestBody 

主要用来处理Content-Type是application/json，application/xml的Json字符串。

多用在Post类型请求中。

### @Pageable

Spring Data库中定义的一个接口，该接口是所有分页相关信息的一个抽象，通过该接口我们可以得到和分页相关所有信息。

其中比较重要的信息包括两个：一个分页的信息，一个是排序的信息。

```yaml
page：第几页，从0页开始，默认为第0也
size：每一页的大小，默认为20
sort：排序相关信息，以property，property(ASC|DESC)的方式组织，例如sort=firstname&sort=lastname，desc表示
```

### @PageableDefault

表示spring data提供给我们个性化设置pageable默认配置。

比如@PageableDefault(value = 15, sort = {"id"}, direction = Sort.Direction.DESC)就表示默认情况下我们按照id倒序排列，每一页的大小为15。

```java

@ResponseBody
@RequestMapping(value = "list", method=RequestMethod.GET)
public Page<blog> listByPageable(@PageableDefault(value = 15, sort = { "id" }, direction = Sort.Direction.DESC) 
    Pageable pageable) {
    return blogRepository.findAll(pageable);
}
```

### @RequestMapping 

主要将HTTP请求映射到MVC和REST控制器的处理方法上，可以具体细分为一下几类：

1. 将特定的请求或者请求模式映射到一个控制器上，甚至还可以具体到方法级别的映射上，参考如下：

```java
@RestController
@RequestMapping("/home")      // /home/*的请求会由IndexController 进行处理
public class IndexController {
    @RequestMapping("/")          // /home的请求会由get()方法来处理
    String get() {
        //mapped to hostname:port/home/
        return "Hello from get";
    }
    @RequestMapping("/index")   // /home/index的请求会由index()来处理
    String index() {
        //mapped to hostname:port/home/index/
        return "Hello from index";
    }
}
```

2. 可以处理多个URL，比如将多个请求映射到一个方法上去，参考如下：

```java
@RestController
@RequestMapping("/home")
public class IndexController {

    @RequestMapping(value = {
        "",
        "/page",
        "page*",
        "view/*,**/msg"
    })
    String indexMultipleMapping() {
        return "Hello from index multiple mapping.";
    }
}
```

在上述代码中，如下URL都会由indexMultipleMapping()来处理：

```yaml
/home
/home/
/home/page
/home/pageabc
/home/view/
/home/view/view
```
3.用@RequestMapping处理HTTP的各种方法，如GET、PUT、POST、DELETE以及PATCH。以POST的方法为例，参考如下：

```java
@RestController
@RequestMapping("/home")
public class IndexController {
    @RequestMapping(value = "/prod", produces = {
        "application/JSON"
    })
    @ResponseBody
    String getProduces() {             //getProduces()方法会产生一个JSON的响应
        return "Produces attribute";
    }

    @RequestMapping(value = "/cons", consumes = {
        "application/JSON",
        "application/XML"
    })
    String getConsumes() {         //getCOnsumes()方法可以同时处理请求中的JSON和XML内容
        return "Consumes attribute";
    }
}
```
@GetMapping和@PutMapping：是@RequestMapping的快捷方式。

比如@GetMapping所扮演的是@RequestMapping(method=RequestMethod.GET)的一个快捷方式。