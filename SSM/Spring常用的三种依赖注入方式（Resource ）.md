---
title: Spring常用的三种依赖注入方式（Resource）
date: 2018-07-08 19:56:24
tags: 
    - Spring
---

Spring 不但支持自己定义的 @Autowired 的注解，还支持几个由 JSR-250 规范定义的注解，它们分别是 @Resource、@PostConstruct 以及 @PreDestroy。

## @Resource
@Resource 的作用相当于 @Autowired，只不过 @Autowired 按 byType 自动注入，面@Resource 默认按 byName 自动注入罢了。

@Resource 有两个属性是比较重要的，分别是 name 和 type，Spring 将@Resource 注解的 name 属性解析为 Bean 的名字，而 type 属性则解析为 Bean 的类型。

所以如果使用 name 属性，则使用 byName 的自动注入策略，而使用 type 属性时则使用 byType 自动注入策略。

如果既不指定 name 也不指定 type 属性，这时将通过反射机制使用 byName 自动注入策略。

Resource 注解类位于 Spring 发布包的 lib/j2ee/common-annotations.jar 类包中，因此在使用之前必须将其加入到项目的类库中。

来看一个使用@Resource的例子（使用 @Resource 注解的 Boss.java）：

```java
package com.baobaotao;

import javax.annotation.Resource;

public class Boss {
    // 自动注入类型为 Car 的 Bean
    @Resource
    private Car car;

    // 自动注入 bean 名称为 office 的 Bean
    @Resource(name = "office")
    private Office office;
}
```
一般情况下，我们无需使用类似于 @Resource(type=Car.class) 的注解方式，因为 Bean 的类型信息可以通过 Java 反射从代码中获取。

## @Resource生效
要让 JSR-250 的注解生效，除了在 Bean 类中标注这些注解外，还需要在 Spring 容器中注册一个负责处理这些注解的 BeanPostProcessor：

```yaml
<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor"/>
```

CommonAnnotationBeanPostProcessor 实现了 BeanPostProcessor 接口，它负责扫描使用了 JSR-250 注解的 Bean，并对它们进行相应的操作。

## @PostConstruct 和 @PreDestroy

Spring 容器中的 Bean 是有生命周期的，Spring 允许在 Bean 在初始化完成后以及 Bean 销毁前执行特定的操作，您既可以通过实现 InitializingBean/DisposableBean 接口来定制初始化之后 / 销毁之前的操作方法，也可以通过 <bean> 元素的 init-method/destroy-method 属性指定初始化之后 / 销毁之前调用的操作方法。

JSR-250 为初始化之后/销毁之前方法的指定定义了两个注解类，分别是 @PostConstruct 和 @PreDestroy，这两个注解**只能应用于方法上**。

标注了 @PostConstruct 注解的方法将在类实例化后调用，而标注了 @PreDestroy 的方法将在类销毁之前调用。

使用 @PostConstruct 和 @PreDestroy 注解的 Boss.java

```java
package com.baobaotao;

import javax.annotation.Resource;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class Boss {
    @Resource
    private Car car;

    @Resource(name = "office")
    private Office office;

    @PostConstruct
    public void postConstruct1(){
        System.out.println("postConstruct1");
    }

    @PreDestroy
    public void preDestroy1(){
        System.out.println("preDestroy1"); 
    }
    …
}
```

您只需要在方法前标注 @PostConstruct 或 @PreDestroy，这些方法就会在 Bean 初始化后或销毁之前被 Spring 容器执行了。

我们知道，不管是通过实现 InitializingBean/DisposableBean 接口，还是通过 <bean> 元素的init-method/destroy-method 属性进行配置，都只能为 Bean 指定**一个**初始化 / 销毁的方法。

但是使用 @PostConstruct 和 @PreDestroy 注解却可以指定**多个**初始化 / 销毁方法，那些被标注 @PostConstruct 或@PreDestroy 注解的方法都会在初始化 / 销毁时被执行。

通过以下的测试代码，您将可以看到 Bean 的初始化 / 销毁方法是如何被执行的：

```java
package com.baobaotao;

import org.springframework.context.support.ClassPathXmlApplicationContext;

public class AnnoIoCTest {

    public static void main(String[] args) {
        String[] locations = {"beans.xml"};
        ClassPathXmlApplicationContext ctx = 
            new ClassPathXmlApplicationContext(locations);
        Boss boss = (Boss) ctx.getBean("boss");
        System.out.println(boss);
        ctx.destroy();// 关闭 Spring 容器，以触发 Bean 销毁方法的执行
    }
}
```
这时，您将看到标注了 @PostConstruct 的 postConstruct1() 方法将在 Spring 容器启动时，创建Boss Bean 的时候被触发执行;

而标注了 @PreDestroy注解的 preDestroy1() 方法将在 Spring 容器关闭前销毁Boss Bean 的时候被触发执行。

## 使用 <context:annotation-config/> 简化配置

Spring 2.1 添加了一个新的 context 的 Schema 命名空间，该命名空间对注解驱动、属性文件引入、加载期织入等功能提供了便捷的配置。

我们知道注解本身是不会做任何事情的，它仅提供元数据信息。要使元数据信息真正起作用，必须让负责处理这些元数据的处理器工作起来。

而我们前面所介绍的 AutowiredAnnotationBeanPostProcessor 和 CommonAnnotationBeanPostProcessor 就是处理这些注解元数据的处理器。

但是直接在 Spring 配置文件中定义这些 Bean 显得比较笨拙。Spring 为我们提供了一种方便的注册这些BeanPostProcessor 的方式，这就是 <context:annotation-config/>。

请看下面的配置：

调整 beans.xml 配置文件:

```yaml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xmlns:context="http://www.springframework.org/schema/context"
     xsi:schemaLocation="http://www.springframework.org/schema/beans 
 http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
 http://www.springframework.org/schema/context 
 http://www.springframework.org/schema/context/spring-context-2.5.xsd">
 
    <context:annotation-config/> 

    <bean id="boss" class="com.baobaotao.Boss"/>
    <bean id="office" class="com.baobaotao.Office">
        <property name="officeNo" value="001"/>
    </bean>
    <bean id="car" class="com.baobaotao.Car" scope="singleton">
        <property name="brand" value=" 红旗 CA72"/>
        <property name="price" value="2000"/>
    </bean>
</beans>
```

<context:annotationconfig/> 将隐式地向 Spring 容器注册
- AutowiredAnnotationBeanPostProcessor
- CommonAnnotationBeanPostProcessor
- PersistenceAnnotationBeanPostProcessor 
- equiredAnnotationBeanPostProcessor

这 4 个 BeanPostProcessor。

在配置文件中使用 context 命名空间之前，必须在 <beans> 元素中声明 context 命名空间。

