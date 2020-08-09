---
title: Spring MVC工作流程
date: 2017-06-15 19:56:24
tags: 
    - Spring MVC
---
<meta name="referrer" content="no-referrer" />

## SpringMVC简介

   SpringMVC是一种基于Spring实现了Web MVC设计模式的请求驱动类型的轻量级Web框架，
使用了MVC架构模式的思想，将web层进行职责解耦，并管理应用所需对象的生命周期，
为简化日常开发，提供了很大便利。

SpringMVC提供了：
- 总开关DispatcherServlet；
- 请求处理映射器(Handler Mapping)和处理适配器（Handler Adapter），
- 视图解析器(View Resolver)进行视图管理；
- 动作处理器Controller接口（包含ModelAndView，以及处理请求响应对象request和response），

配置灵活，支持文件上传，数据简单转化等强大功能。

## 工作流程与介绍

![](https://img-blog.csdn.net/20180803145042704?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxOTEyMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**流程解析：**
```yaml
（1）用户发起请求到前端控制器（DispatcherServlet），该控制器会过滤出哪些请求可以访问Servlet、哪些不能访问。就是url-pattern的作用，并且会加载springmvc.xml配置文件。
（2）前端控制器会找到处理器映射器（HandlerMapping），通过HandlerMapping完成url到controller映射的组件，简单来说，就是将在springmvc.xml中配置的或者注解的url与对应的处理类找到并进行存储，用map<url,handler>这样的方式来存储。
（3）HandlerMapping有了映射关系，并且找到url对应的处理器，HandlerMapping就会将其处理器（Handler）返回，在返回前，会加上很多拦截器。
（4）DispatcherServlet拿到Handler后，找到HandlerAdapter（处理器适配器），通过它来访问处理器，并执行处理器。
（5）执行处理器
（6）处理器会返回一个ModelAndView对象给HandlerAdapter
（7）通过HandlerAdapter将ModelAndView对象返回给前端控制器(DispatcherServlet)
（8）前端控制器请求视图解析器(ViewResolver)去进行视图解析，根据逻辑视图名解析成真正的视图(jsp)，其实就是将ModelAndView对象中存放视图的名称进行查找，找到对应的页面形成视图对象
（9）返回视图对象到前端控制器。
（10）视图渲染，就是将ModelAndView对象中的数据放到request域中，用来让页面加载数据的。
（11）通过第8步，通过名称找到了对应的页面，通过第10步，request域中有了所需要的数据，那么就能够进行视图渲染了。最后将其返回即可。
```
```yaml
简而言之，SpringMVC通过DispatcherServlet这个前端控制器(也叫中央调度器，我认为中央调度器更能体现其作用)，来调用mvc的三大件:Controller、Model、View。
这样就保证MVC的每一个组件只与DispatcherServlet耦合，而彼此之间独立运行，大大降低了程序的耦合性。
```


## SpringMVC这个框架时如何实现MVC模式的?

```yaml
注意SpringMVC中并没有涉及有关于Controller接口规范的实现,SpringMVC是通过调用Handler来实现Controller这一层的。
SpringMVC使用了适配器模式，前端控制器使用HandlerAdapter来调用不同的Controller,然后才是Controller调用Model产生数据模型; 
产生的数据模型将会再次返回到前端控制器，并由前端控制器决定使用不同的模板引擎将页面进行渲染。
```


