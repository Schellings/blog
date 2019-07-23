## 引言
在mybatis的基础知识中我们已经可以对mybatis的工作方式窥斑见豹。但是，为什么还要要学习mybatis的工作原理？

因为，随着mybatis框架的不断发展，如今已经越来越趋于自动化，从代码生成，到基本使用，我们甚至不需要动手写一句SQL就可以完成一个简单应用的全部CRUD操作。

从原生mybatis到mybatis-spring，到mybatis-plus再到mybatis-plus-spring-boot-starter。

spring在发展，mybatis同样在随之发展。

万变的外表终将迷惑人们的双眼，只要抓住核心我们永远不会迷茫！

## 工作原理原型图

![工作原理原型图](https://img-blog.csdn.net/20180624002302854?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ3NDUwNjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 工作原理解析

mybatis应用程序通过SqlSessionFactoryBuilder从mybatis-config.xml配置文件（也可以用Java文件配置的方式，需要添加@Configuration）中构建出SqlSessionFactory（SqlSessionFactory是线程安全的）；

然后，SqlSessionFactory的实例直接开启一个SqlSession，再通过SqlSession实例获得Mapper对象并运行Mapper映射的SQL语句，完成对数据库的CRUD和事务提交，之后关闭SqlSession。

**说明：** SqlSession是单线程对象，因为它是非线程安全的，是持久化操作的独享对象，类似jdbc中的Connection，底层就封装了jdbc连接。

## 详细流程

1、加载mybatis全局配置文件（数据源、mapper映射文件等），解析配置文件，MyBatis基于XML配置文件生成Configuration，和一个个MappedStatement（包括了参数映射配置、动态SQL语句、结果映射配置），其对应着<select | update | delete | insert>标签项。

2、SqlSessionFactoryBuilder通过Configuration对象生成SqlSessionFactory，用来开启SqlSession。

3、SqlSession对象完成和数据库的交互：

a、用户程序调用mybatis接口层api（即Mapper接口中的方法）

b、SqlSession通过调用api的Statement ID找到对应的MappedStatement对象

c、通过Executor（负责动态SQL的生成和查询缓存的维护）将MappedStatement对象进行解析，sql参数转化、动态sql拼接，生成jdbc Statement对象

d、JDBC执行sql。

e、借助MappedStatement中的结果映射关系，将返回结果转化成HashMap、JavaBean等存储结构并返回。

## mybatis层次图

![mybatis层次图](https://img-blog.csdn.net/20180625095624918?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQ3NDUwNjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)