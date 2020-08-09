---
title: Spring MVC常见面试问题
date: 2017-06-18 19:56:24
tags: 
    - Spring MVC
---
<meta name="referrer" content="no-referrer" />

## 什么是Spring MVC?作用？
```yaml
springMVC是一种web层mvc框架，用于替代servlet（处理|响应请求，获取表单参数，表单校验等）。
```

## 为什么要用Spring MVC?

```yaml
基本上，框架的作用就是用来简化编程的，相对于servlet来说，获取表单参数,响应请求等变得更简单了。
```
## spring mvc底层执行流程(工作原理)

![Spring MVC工作流程图](https://ss0.baidu.com/6ONWsjip0QIZ8tyhnq/it/u=2256951759,4246877682&fm=173&app=49&f=JPEG?w=640&h=382&s=71C6FC12CF304D8800D9D15E03007071)

## 说说spring mvc中常用注解有哪些，分别什么作用？

- @Controller

标识这个类是一个控制器

- @RequestMapping

给控制器方法绑定一个uri

- @ResponseBody

将java对象转成json，并且发送给客户端

- @RequestBody

将客户端请求过来的json转成java对象

- @RequestParam

当表单参数和方法形参名字不一致时，做一个名字映射

- @PathVarible

用于获取uri中的参数,比如user/1中1的值

Rest风格的新api

- @RestController

@Controller+@ResponseBody

- @GetMapping@DeleteMapping@PostMapping

- @PutMapping


其他注解

- @SessionAttribute

声明将什么模型数据存入session

- @CookieValue

获取cookie值

- @ModelAttribute

将方法返回值存入model中

- @HeaderValue

获取请求头中的值

## spring mvc和strus2的区别？
（1）入口不同:

spring mvc 入口是Servlet。struts2入口是filter。

（2）生命周期不同:

spring mvc Controller是单例的。所以不能使用成员变量获取参数。所以效率高。

struts action是多例的。所以可以使用成员变量获取参数。所以效率低。

## 如何在spring mvc实现RESTful服务
（1）导入jackson2包

（2）开启注解驱动 <mvc:annotation-driven/>

（3）json交互：@RequestBody @ResponseBody

## spring mvc如何返回JSON数据

在处理方法前加上 @ResponseBody注解

或者

在控制器上使用 @RestController

## 什么是拦截器？有什么用？spring mvc如何定义拦截器？

what:

类似于filter的一个对象，用于预处理以及后处理处理器(控制器)。

how:

新建class实现HandlerInterceptor重写三个方法preHandler postHandler afterCompletion springmvc.xml中配置拦截器。

## spring mvc中如何做表单数据校验？

环境搭建

（1）springmvc.xml中配置一个validator

（2）<mvc:annotation-driven validator="validator"/>;

给Entity添加校验规则

@NotEmpty

@Length

...

用BindingResult 紧接着entity之后来接收错误信息。

test(User user,BindingResult rs)