---
title: Spring Boot 一些常用的注解介绍（一）
date: 2018-01-26 19:56:24
tags: Spring Boot
---
## Spring Boot 一些常用的注解介绍（一）

### @SpringBootApplication(scanBasePackages = {“com.qiu”}, exclude = {})

```yaml
包含了@ComponentScan、@Configuration和@EnableAutoConfiguration注解。其中@ComponentScan让spring Boot扫描到Configuration类并把它加入到程序上下文。

scanBasePackages：设置ComponentScan扫描包路径。
```

### @ComponentScan
```yaml
组件扫描，可自动发现和装配一些Bean。

@ComponentScan告诉Spring哪个packages的用注解标识的类会被spring自动扫描并且装入bean容器。 

例如，如果你有个类用@Controller注解标识了，那么，如果不加上@ComponentScan，自动扫描该controller。

那么该Controller就不会被spring扫描到，更不会装入spring容器中，因此你配置的这个Controller也没有意义。
```

### @EnableAutoConfiguration

```yaml
自动配置。

@EnableAutoConfiguration和@Configuration是成对出现，@EnableAutoConfiguration负责去扫描带有@Configuration的类。 

由于spring boot相当于一个CI，可以持续集成，所有@EnableAutoConfiguration相当于对集成进来的模块进行初始化的工作。 

会去扫描pom文件中所依赖的jar包，依赖了哪个jar包就对这个jar进行初始化。
```
### @Configuration

```yaml
等同于spring的XML配置文件；使用Java代码可以检查类型安全。可理解为用spring的时候xml里面的标签。
```

### @Bean
```yaml
可理解为用spring的时候xml里面的标签
```
### @RestController
```yaml
注解是@Controller和@ResponseBody的合集,表示这是个控制器bean,并且是将函数的返回值直接填入HTTP响应体中,是REST风格的控制器。
```


### @Value("${application.username:defaultValue"}
```yaml
使用@value注解，从application.properties配置文件读取值，没读取到就用默认值defaultValue
```

### @MapperScan("cn.qiu.mapper")
```yaml
mybatis框架中的dao扫描
```


### @ServletComponentScan(basePackages = { "cn.qiu"})
```yaml
扫描工程中的Servlet、Filter、Listener（带注解的）
```
### @RunWith(SpringJUnit4ClassRunner.class)
```yaml
SpringJUnit支持，由此引入Spring-Test框架支持！
```
### @SpringApplicationConfiguration(classes = App.class)
```yaml
指定我们SpringBoot工程的Application启动类（App是项目的启动类）
```

### @WebAppConfiguration
```yaml
由于是Web项目，Junit需要模拟ServletContext，因此我们需要给我们的测试类加上@WebAppConfiguration。
```