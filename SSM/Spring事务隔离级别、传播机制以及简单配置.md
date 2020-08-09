---
title: Spring事务隔离级别、传播机制以及简单配置
date: 2017-07-08 19:56:24
tags: 
    - Spring
---
<meta name="referrer" content="no-referrer" />

## Spring支持的事务声明方式

1.  编程式事务  当系统需要明确的，细粒度的控制各个事务的边界，应选择编程式事务。

2.  声明式事务  当系统对于事务的控制粒度较粗时，应该选择申明式事务，通过<tx>标签和<aop>切面形式在xml中进行配置。

3.  无论你选择上述何种事务方式去实现事务控制，spring都提供基于门面设计模式的事务管理器供选择，如下是spring事务中支持的事务管理器

事务管理器实现(org.springframework.*)|使用时机
---|---
jdbc.datasource.DataSourceTransactionManager|使用jdbc的抽象以及ibatis支持
orm.hibernate.HibernateTransactionManager|使用hibernate支持(默认3.0以下版本)
orm.hibernate3.HibernateTransactionManager|使用hibernate3支持
transaction.jta.JtaTransactionManager|使用分布式事务（分布式数据库支持） 
orm.jpa.JpaTransactionManager|使用jpa做为持久化工具
orm.toplink.TopLinkTransactionManager|使用TopLink持久化工具
orm.jdo.JdoTransactionManager|使用Jdo持久化工具
jms.connection.JmsTransactionManager|使用JMS 1.1+
jms.connection.JmsTransactionManager102|使用JMS 1.0.2
transaction.jta.OC4JJtaTransactionManager|使用oracle的OC4J JEE容器
transaction.jta.WebLogicJtaTransactionManager|在weblogic中使用分布式数据库
jca.cci.connection.CciLocalTransactionManager|使用jrping对J2EE Connector Architecture (JCA)和Common Client Interface (CCI)的支持

配置示例如下:

```yaml
<bean id="transactionManager"class="org.springframework.jdbc.datasource.DataSourceTransactionManager"> 
  <propertyname="dataSource" ref="dataSource"/>  //引入数据源
</bean>
```

## Spring支持7种事务传播行为

传播行为|含义
---|---
propagation_required（xml文件中为required)|表示当前方法必须在一个具有事务的上下文中运行，如有客户端有事务在进行，那么被调用端将在该事务中运行，否则的话重新开启一个事务。（如果被调用端发生异常，那么调用端和被调用端事务都将回滚）
propagation_supports(xml文件中为supports）|表示当前方法不必需要具有一个事务上下文，但是如果有一个事务的话，它也可以在这个事务中运行
propagation_mandatory(xml文件中为mandatory）|表示当前方法必须在一个事务中运行，如果没有事务，将抛出异常
propagation_nested(xml文件中为nested)|表示如果当前方法正有一个事务在运行中，则该方法应该运行在一个嵌套事务中，被嵌套的事务可以独立于被封装的事务中进行提交或者回滚。如果封装事务存在，并且外层事务抛出异常回滚，那么内层事务必须回滚，反之，内层事务并不影响外层事务。如果封装事务不存在，则同propagation_required的一样
propagation_never（xml文件中为never）|表示当方法务不应该在一个事务中运行，如果存在一个事务，则抛出异常
propagation_requires_new(xml文件中为requires_new）|表示当前方法必须运行在它自己的事务中。一个新的事务将启动，而且如果有一个现有的事务在运行的话，则这个方法将在运行期被挂起，直到新的事务提交或者回滚才恢复执行。
propagation_not_supported（xml文件中为not_supported）|表示该方法不应该在一个事务中运行。如果有一个事务正在运行，他将在运行期被挂起，直到这个事务提交或者回滚才恢复执行

## Spring中的事务隔离级别

隔离级别|含义
---|---
isolation_default|使用数据库默认的事务隔离级别
isolation_read_uncommitted|允许读取尚未提交的修改，可能导致脏读、幻读和不可重复读
isolation_read_committed|允许从已经提交的事务读取，可防止脏读、但幻读，不可重复读仍然有可能发生
isolation_repeatable_read|对相同字段的多次读取的结果是一致的，除非数据被当前事务自生修改。可防止脏读和不可重复读，但幻读仍有可能发生
isolation_serializable|完全服从acid隔离原则，确保不发生脏读、不可重复读、和幻读，但执行效率最低。

## 脏读、不可重复读、幻象读概念说明
```yaml
a.脏读：指当一个事务正字访问数据，并且对数据进行了修改，而这种数据还没有提交到数据库中。
这时，另外一个事务也访问这个数据，然后使用了这个数据。因为这个数据还没有提交那么另外一个事务读取到的这个数据我们称之为脏数据。
依据脏数据所做的操作肯能是不正确的。
```
```yaml
b.不可重复读：指在一个事务内，多次读同一数据。在这个事务还没有执行结束，另外一个事务也访问该同一数据，那么在第一个事务中的两次读取数据之间，由于第二个事务的修改第一个事务两次读到的数据可能是不一样的。
这样就发生了在一个事物内两次连续读到的数据是不一样的，这种情况被称为是不可重复读。
```
```yaml
c.幻象读：一个事务先后读取一个范围的记录，但两次读取的纪录数不同，我们称之为幻象读（两次执行同一条 select 语句会出现不同的结果，第二次读会增加一数据行，并没有说这两次执行是在同一个事务中）
```

## Spring事务只读属性
Spring事务只读的含义是指，如果后端数据库发现当前事务为只读事务，那么就会进行一系列的优化措施。

它是在后端数据库进行实施的，因此，只有对于那些有可能启动一个新事务的传播行为（REQUIRED,REQUIRES_NEW,NESTED）的方法来说，才有意义。

（测试表明，当使用JDBC事务管理器并设置当前事务为只读时，并不能发生预期的效果，即能执行删除，更新，插入操作）

## Spring的事务超时

有的时候为了系统中关键部分的性能问题，它的事务执行时间应该尽可能的短。因此可以给这些事务设置超时时间，以秒为单位。

我们知道事务的开始往往都会发生数据库的表锁或者被数据库优化为行锁，如果允许时间过长，那么这些数据会一直被锁定，影响系统的并发性。

因为超时时钟是在事务开始的时候启动，因此只有对于那些有可能启动新事物的传播行为（REQUIRED,REQUIRES_NEW,NESTED）的方法来说，事务超时才有意义。
 
 
## 事务回滚规则

Spring中可以指定当方法执行并抛出异常的时候，哪些异常回滚事务，哪些异常不回滚事务。

默认情况下，只在方法抛出运行时异常的时候才回滚(runtime exception)。

而在出现受阻异常(checked exception)时不回滚事务。当然可以采用申明的方式指定哪些受阻异常像运行时异常那样指定事务回滚。

## 单个方法的事务传播机制(注意这里说的是单个方法）

1、整备工作: spring申明式事务配置,将aop，tx命名空间添加到当前的Spring配置文件头中，这里不在累述。

2、定义一个事务AOP通知(add开头名单方法)
```yaml
<tx:advice id="txAdvice" transactionManager="transactionManager"> 
    <tx:attributes> 
      <tx:methodname="add*" propagation="REQUIRED"/> 
    </tx:attributes> 
</tx:advice>
```
3、定义一个事务切面，即应该在哪些类的哪些方法上面进行事务切入，多个包体可以使用||隔开

```yaml
<aop:config> 
    <aop:advisor pointcut="execution(* *..zx.spring.UserService*.*(..))||execution(**..spring.ServiceFacade.*(..))||execution(**..spring.BookService.*(..))" 
        advice-ref="txAdvice"/> 
</aop:config>
```
4、事务通知中Propagation设置为7中传播行为的情况如下(单方法):

@1、方法的事务传播机制为REQUIRED， REQUIRES_NEW，并且抛出运行时异常时，将回滚事务。当方法抛出受检查的异常时，将不会回滚事务。

@2、方法的事务传播机制为MANDATORY，由于单个方法执行没有指定任何事务传播机制，因此抛出异常。

@3、方法的事务传播机制为NESTED，由于NESTED内嵌事务，如果没有外层事务，则新建一个事务，行为同REQUIRED一样。

@4、方法的事务传播机制为NEVER，由于发生运行时异常，事务本应该回滚，但是事务传播机制为NEVER，因此找不到事务进行回滚。

@5、方法的事务传播机制为NOT_SUPPORTED，单个方法被执行时，不会创建事务。如果当前有事务，将封装事务挂起，知道该方法执行完成再恢复封装事务，继续执行。

@6、方法的事务传播机制为SUPPORTS，单个方法 调用时supports行为同NEVER一样，不会创建事务，只是如果有事务，则会加入到当前事务中去。

## 多个方法间的事务传播机制(注意这里说的是多个方法)

1、整备工作: spring申明式事务配置,将aop，tx命名空间添加到当前的spring配置文件头中，不在累述。

2、定义一个事务AOP通知(add开头名单方法)

```yaml
<tx:advice id="txAdvice" transactionManager="transactionManager"> 
     <tx:attributes> 
         <tx:methodname="add*" propagation="REQUIRED"/>
                 <tx:methodname="update*" propagation="REQUIRED"/> 
     </tx:attributes> 
</tx:advice>
```
3、定义一个事务切面，即应该在哪些类的哪些方法上面进行事务切入

```yaml
<aop:config> 
    <aop:advisor pointcut="execution(**..zx.spring.UserService*.*(..))||execution(**..spring.ServiceFacade.*(..))||execution(**..spring.BookService.*(..))"
        advice-ref="txAdvice"/> 
</aop:config>
```


4、事务通知中Propagation设置为7中传播行为的情况如下(多方法间传递):

@1、多个方法的事务传播机制都为REQUIRED，由于REQUIRED方式是如果有一个事务，则加入事务中，如果没有，则新建一个事务，也就是说方法1一开始就会创建一个事务，而方法2发现有事务存在则会加入到事务中，方法2抛出运行时异常,整个事务将会回滚。如果方法2抛出的是可捕获异常，事务将不起作用，方法1执行成功插入，方法2异常中断。或者我们将方法2设置rollback-for属性，那么在指定的异常之内事务都将回滚

@2、方法1的事务传播机制为REQUIRED，方法2的事务传播机制为NEVER，由于方法1会创建一个事务，而方法2是NEVER,不应该在事务中运行，因此导致了冲突会报错。Existingtransaction found for transaction marked with propagation 'never'

@3、方法1的事务传播机制为REQUIRED，方法2的事务传播机制为MANDATORY，执行机制和@1一致。

@4、方法1的事务传播机制为REQUIRED，方法2的事务传播机制为NESTED，内嵌事务，也就是方法1执行体内放入方法2的调用，由于方法2的事务传播机制为NESTED内嵌事务，因此，开始一个新的事务。并且方法2内部抛出RuntimeException,因此内部嵌套事务回滚到外层事务创建的保存点。

**注意这个地方，我们抛出的是运行时异常，如果我们抛出受检查的异常，那么spring会默认的忽略此异常。**

如果内层事务抛出检查异常，那么外层事务将忽略此异常，但是会产生一个问题。那就是：外层事务使用jdbc的保存点API来实现嵌套事务，

**但是数据库不一定支持。** ：可能会抛出如下异常java.sql.SQLException:不支持的特性。

总结: 对于NESTED内层事务而言，内层事务独立于外层事务，可以独立递交或者回滚,如果内层事务抛出的是运行异常，外层事务进行回滚，内层事务也会进行回滚。

@5、方法1的事务传播机制为REQUIRED，方法2的事务传播机制为REQUIRES_NEW。(情况比较复杂)

情况一，方法2不抛出任何异常下:方法1先创建事务，但是方法2也要运行在自己的事务中，因此也会创建一个事务，方法1的事务将会挂起，直到方法2的事务执行完成才会接着事务的提交完成

情况二，方法2抛出运行时异常下: 对于REQUIRES_NEW事务传播机制，如果被调用端(方法2)抛出运行时异常，则被调用端事务回滚，如果调用段代码(方法1)捕获了被调用端抛出的运行时异常，那么调用端事务提交，不回滚。反之，如果调用段代码未捕获被调用端抛出的运行时异常，那么调用端事务回滚，不提交。

@6、方法1的事务传播机制为REQUIRED，方法2的事务传播机制为SUPPORTS，方法1创建一个事务，方法2加入到当前事务中，方法2抛出运行异常将进行事务的回滚，如果方法1不具有事务，由于方法2的特性是事务环境可有可无，方法2也不会运行在事务中，所以也不会进行回滚。如果方法2抛出的是受检查异常，异常将会忽略，事务将会提交，当然还有别的情况没有一一验证

@7、方法1的事务传播机制为REQUIRED，方法2的事务传播机制为NOT_SUPPORTED，由于方法2事务特性，方法1的事务将会被挂起，方法2抛出运行时异常，但是方法2是不在事务环境中运行的，所以回滚不了，而方法1中未捕获此异常，将进行事务回滚，如果方法2抛出的是运行异常，即使方法1没有捕获此异常，也将被忽略，不进行事务的回滚。

## Spring声明式事务配置

### 基于@Transactional注解(推荐配置):

```yaml
<!-- 事务管理器，对mybatis操作数据库进行事务控制，spring使用jdbc的事务控制类  -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSourceSuport"></property>
</bean>
<!—开启spring 事务注解支持  mode="aspectj"表示采用切面    mode="proxy"表示代理模式(默认) -->
<tx:annotation-driven transaction-manager="transactionManager"  />  

```

MyBatis自动参与到spring事务管理中，无需额外配置，只要org.mybatis.spring.SqlSessionFactoryBean引用的数据源与DataSourceTransactionManager引用的数据源一致即可，否则事务管理会不起作用。

@Transactional可以作用于接口、接口方法、类以及类方法上。

当作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。 

虽然@Transactional 注解可以作用于接口、接口方法、类以及类方法上，但是 Spring 建议不要在接口或者接口方法上使用该注解，因为这只有在使用基于接口的代理时它才会生效。

另外，**@Transactional 注解应该只被应用到public 方法上**，这是由 Spring AOP 的本质决定的。

如果你在 protected、private 或者默认可见性的方法上使用@Transactional 注解，**这将被忽略**，也不会抛出任何异常。

默认情况下，**只有来自外部的方法调用才会被AOP代理捕获**，也就是，类内部方法调用本类内部的其他方法并不会引起事务行为，即使被调用方法使用@Transactional注解进行修饰。


