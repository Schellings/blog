---
title: Spring常用的三种依赖注入方式（@Autowired）
date: 2017-07-05 19:56:24
tags: 
    - Spring
---
<meta name="referrer" content="no-referrer" />
## 概述
平常的java开发中，程序员在某个类中需要依赖其它类的方法，则通常是new一个依赖类再调用类实例的方法，这种开发存在的问题是new的类实例不好统一管理，spring提出了依赖注入的思想，即依赖类不由程序员实例化，而是通过spring容器帮我们new指定实例并且将实例注入到需要该对象的类中。

依赖注入的另一种说法是“控制反转”，通俗的理解是：平常我们new一个实例，这个实例的控制权是我们程序员，而控制反转是指new实例工作不由我们程序员来做而是交给spring容器来做。Spring通过DI（依赖注入）实现IOC（控制反转）。

常用的注入方式主要有三种：**构造方法注入，set方法参数注入，接口注入**。

注解配置相对于XML 配置具有很多的优势：

它可以充分利用 Java 的反射机制获取类结构信息，这些信息可以有效减少配置的工作。

如使用 JPA 注解配置 ORM 映射时，我们就不需要指定 PO 的属性名、类型等信息，如果关系表字段和 PO 属性名、类型都一致，您甚至无需编写任务属性映射信息——因为这些信息都可以通过 Java 反射机制获取。

注解和 Java 代码位于一个文件中，而 XML 配置采用独立的配置文件，大多数配置信息在程序开发完成后都不会调整，如果配置信息和 Java 代码放在一起，有助于增强程序的内聚性。

而采用独立的 XML 配置文件，程序员在编写一个功能时，往往需要在程序文件和配置文件中不停切换，这种思维上的不连贯会降低开发效率。

在这篇文章里，我们将向您讲述使用注解进行 Bean 定义和依赖注入的内容。

## 原来我们是怎么做的

在使用注解配置之前，先来回顾一下传统上是如何配置 Bean 并完成 Bean 之间依赖关系的建立。

下面是 3 个类，它们分别是 Office、Car 和 Boss，这 3 个类需要在 Spring 容器中配置为 Bean：

Office.java（Office 仅有一个属性）：
```java
package com.baobaotao;
public class Office {
    private String officeNo =”001”;

    //省略 get/setter

    @Override
    public String toString() {
        return "officeNo:" + officeNo;
    }
}
```
Car.java（Car 拥有两个属性）:

```java
package com.baobaotao;

public class Car {
    private String brand;
    private double price;

    // 省略 get/setter

    @Override
    public String toString() {
        return "brand:" + brand + "," + "price:" + price;
    }
}
```

Boss.java（Boss 拥有 Office 和 Car 类型的两个属性）：

```java
package com.baobaotao;

public class Boss {
    private Car car;
    private Office office;

    // 省略 get/setter

    @Override
    public String toString() {
        return "car:" + car + "\n" + "office:" + office;
    }
}
```

我们在 Spring 容器中将 Office 和 Car 声明为 Bean，并注入到 Boss Bean 中：下面是使用传统 XML 完成这个工作的配置文件 beans.xml：

beans.xml 将以上三个类配置成 Bean:

```yaml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
 http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">
    <bean id="boss" class="com.baobaotao.Boss">
        <property name="car" ref="car"/>
        <property name="office" ref="office" />
    </bean>
    <bean id="office" class="com.baobaotao.Office">
        <property name="officeNo" value="002"/>
    </bean>
    <bean id="car" class="com.baobaotao.Car" scope="singleton">
        <property name="brand" value=" 红旗 CA72"/>
        <property name="price" value="2000"/>
    </bean>
</beans>
```

当我们运行以下代码时，控制台将正确打出 boss 的信息：

测试类：AnnoIoCTest.java：

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
public class AnnoIoCTest {

    public static void main(String[] args) {
        String[] locations = {"beans.xml"};
        ApplicationContext ctx = 
		    new ClassPathXmlApplicationContext(locations);
        Boss boss = (Boss) ctx.getBean("boss");
        System.out.println(boss);
    }
}
```

这说明 Spring 容器已经正确完成了 Bean 创建和装配的工作。

## 使用 @Autowired 注解

Spring 2.5 引入了 @Autowired 注解，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。

来看一下使用@Autowired 进行成员变量自动注入的代码：

使用 @Autowired 注解的 Boss.java：

```java
package com.baobaotao;
import org.springframework.beans.factory.annotation.Autowired;

public class Boss {

    @Autowired
    private Car car;

    @Autowired
    private Office office;

    …
}
```

Spring 通过一个 BeanPostProcessor 对 @Autowired 进行解析，所以要让@Autowired 起作用必须事先在 Spring 容器中声明AutowiredAnnotationBeanPostProcessor Bean。

让 @Autowired 注解工作起来:

```yaml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans 
 http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <!-- 该 BeanPostProcessor 将自动起作用，对标注 @Autowired 的 Bean 进行自动注入 -->
    <bean class="org.springframework.beans.factory.annotation.
        AutowiredAnnotationBeanPostProcessor"/>

    <!-- 移除 boss Bean 的属性注入配置的信息 -->
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
这样，当 Spring 容器启动时，AutowiredAnnotationBeanPostProcessor 将扫描 Spring 容器中所有 Bean。

当发现 Bean 中拥有@Autowired 注解时就找到和其匹配（默认按类型匹配）的 Bean，并注入到对应的地方中去。

按照上面的配置，Spring 将直接采用 Java **反射** 机制对 Boss 中的 car 和 office 这两个私有成员变量进行自动注入。所以对成员变量使用@Autowired后，您大可将它们的 setter 方法（setCar() 和 setOffice()）从 Boss 中删除。

当然，您也可以通过 @Autowired 对方法或构造函数进行标注，来看下面的代码：

将 @Autowired 注解标注在 Setter 方法上：
```java
package com.baobaotao;

public class Boss {
    private Car car;
    private Office office;

     @Autowired
    public void setCar(Car car) {
        this.car = car;
    }
 
    @Autowired
    public void setOffice(Office office) {
        this.office = office;
    }
    …
}
```

这时，@Autowired 将查找被标注的方法的入参类型的 Bean，并调用方法自动注入这些 Bean。而下面的使用方法则对构造函数进行标注：

将 @Autowired 注解标注在构造函数上:

```java
package com.baobaotao;

public class Boss {
    private Car car;
    private Office office;
 
    @Autowired
    public Boss(Car car ,Office office){
        this.car = car;
        this.office = office ;
    }
 
    …
}
```
由于 Boss() 构造函数有两个入参，分别是 car 和 office，@Autowired 将分别寻找和它们类型匹配的 Bean，将它们作为Boss(Car car ,Office office)的入参来创建 Boss Bean。

## 当候选 Bean 数目不为 1 时的应对方法

在默认情况下使用 @Autowired 注解进行自动注入时，Spring 容器中匹配的候选 Bean 数目必须有且仅有一个。

当找不到一个匹配的 Bean 时，Spring 容器将抛出BeanCreationException 异常，并指出必须至少拥有一个匹配的 Bean。我们可以来做一个实验：

候选 Bean 数目为 0 时:
```yaml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
     xsi:schemaLocation="http://www.springframework.org/schema/beans 
 http://www.springframework.org/schema/beans/spring-beans-2.5.xsd ">
 
    <bean class="org.springframework.beans.factory.annotation.
        AutowiredAnnotationBeanPostProcessor"/> 

    <bean id="boss" class="com.baobaotao.Boss"/>

    <!-- 将 office Bean 注解掉 -->
    <!-- <bean id="office" class="com.baobaotao.Office">
    <property name="officeNo" value="001"/>
    </bean>-->

    <bean id="car" class="com.baobaotao.Car" scope="singleton">
        <property name="brand" value=" 红旗 CA72"/>
        <property name="price" value="2000"/>
    </bean>
</beans>
```
由于 office Bean 被注解掉了，所以 Spring 容器中将没有类型为 Office 的 Bean 了，而 Boss 的office 属性标注了 @Autowired，当启动 Spring 容器时，异常就产生了。

当不能确定 Spring 容器中一定拥有某个类的 Bean 时，可以在需要自动注入该类 Bean 的地方可以使用 @Autowired(required = false)，这等于告诉 Spring：在找不到匹配 Bean 时也不报错。来看一下具体的例子：

使用 @Autowired(required = false):

```java
package com.baobaotao;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Required;

public class Boss {

    private Car car;
    private Office office;

    @Autowired
    public void setCar(Car car) {
        this.car = car;
    }
    @Autowired(required = false)
    public void setOffice(Office office) {
        this.office = office;
    }
    …
}
```

当然，一般情况下，使用 @Autowired 的地方都是需要注入 Bean 的，使用了自动注入而又允许不注入的情况一般仅会在开发期或测试期碰到（如为了快速启动 Spring 容器，仅引入一些模块的 Spring 配置文件），所以@Autowired(required = false) 会很少用到。

和找不到一个类型匹配 Bean 相反的一个错误是：如果 Spring 容器中拥有多个候选 Bean，Spring 容器在启动时也会抛出 BeanCreationException 异常。来看下面的例子：

在 beans.xml 中配置两个 Office 类型的 Bean:

```yaml
… 
<bean id="office" class="com.baobaotao.Office">
    <property name="officeNo" value="001"/>
</bean>
<bean id="office2" class="com.baobaotao.Office">
    <property name="officeNo" value="001"/>
</bean>
…
```

我们在 Spring 容器中配置了两个类型为 Office 类型的 Bean，当对 Boss 的 office 成员变量进行自动注入时，Spring 容器将无法确定到底要用哪一个 Bean，因此异常发生了。

Spring 允许我们通过 @Qualifier 注解指定注入 Bean 的名称，这样歧义就消除了，可以通过下面的方法解决异常：

```java
@Autowired
public void setOffice(@Qualifier("office")Office office) {
    this.office = office;
}
```

@Qualifier("office") 中的 office 是 Bean 的名称，所以 @Autowired 和@Qualifier 结合使用时，自动注入的策略就从 byType 转变成 byName 了。

@Autowired 可以对成员变量、方法以及构造函数进行注解，而@Qualifier 的标注对象是成员变量、方法入参、构造函数入参。

正是由于注解对象的不同，所以 Spring 不将 @Autowired 和@Qualifier 统一成一个注解类。

下面是对成员变量和构造函数入参进行注解的代码：

对成员变量进行注解（对成员变量使用 @Qualifier 注解）：

```java
public class Boss {
    @Autowired
    private Car car;
 
    @Autowired
    @Qualifier("office")
    private Office office;
    …
}
```

对构造函数入参进行注解（对构造函数变量使用 @Qualifier 注解）：

```yaml
public class Boss {
    private Car car;
    private Office office;

    @Autowired
    public Boss(Car car , @Qualifier("office")Office office){
        this.car = car;
        this.office = office ;
	}
}
```
@Qualifier 只能和 @Autowired 结合使用，是对 @Autowired 有益的补充。

一般来讲，@Qualifier 对方法签名中入参进行注解会降低代码的可读性，而对成员变量注解则相对好一些。

**使用 JSR-250 的注解**:

Spring 不但支持自己定义的 @Autowired 的注解，还支持几个由 JSR-250 规范定义的注解，它们分别是 @Resource、@PostConstruct 以及 @PreDestroy。


