---
title: Mybatis基础
date: 2017-05-21 19:56:24
tags: 
    - Mybatis
---
## Mybatis初识
MyBatis是一个比较优秀的、开源的数据持久层框架，它可以在实体类与SQL语句之间建立映射关系，替开发人员完成了JavaBean组件与数据库记录实体之间的转化，是一种半自动化的ORM实现。

它内部封装了通过JDBC访问数据库的操作，支持普通SQL查询、存储过程和高级映射，几乎消除了所有的JDBC代码和参数的手工设置，以及结果集的检索。

MyBatis作为持久层框架，其主要思想是将程序中大量的SQL语句分离出来，配置在配置文件中，通过读取配置文件执行相应的SQL语句。

此框架性能可观，小巧易学，应用也比较广泛。

## 依赖配置
添加依赖即可，jar包或pom依赖：
```yaml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```
## SqlSessionFactory
### 什么是SqlSessionFactory？
SqlSessionFactory是MyBatis的核心类，它是单个数据库映射关系经过编译后的内存镜像。

主要功能是提供用于操作数据库的SqlSession。

### 构建SqlSessionFactory？

SqlSessionFactory实例可以通过**SqlSessionFactoryBuilder**获得。

后者可以从一个xml配置文件（mybatis-config.xml文件）或预定制的Configuration实例中构建出SqlSessionFactory。

每一个MyBatis应用都以一个SqlSessionFactory实例为核心，同时SqlSessionFactory也是线程安全的。

SqlSessionFactory一旦被创建，应该在应用执行期间都存在于内存中，应用运行期间不要重复创建多次，**建议使用单例模式**。

xml配置文件中包含了对mybatis系统的核心设置，包括获取数据库连接实例的数据源DataSource和决定事务作用域和控制方式的事务管理器TransactionManager。

一个简单的mybatis-config.xml（未与spring整合）的例子如下：

```yaml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC"/>
      <dataSource type="POOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```
上面的配置信息是关键的部分。xml头部声明用来验证xml文档正确性。

environment元素体中包含了事务管理和连接池的配置。

mappers元素则是包含一组mapper映射器（这些mapper的xml文件包含了SQL代码和映射定义信息）

构建（如果使用像spring这样的框架，或者更高级一点的spring boot，这些工作全都不用我们来做了，开箱即用）：

```java
SqlSessionFactory ssf = new SqlSessionFactoryBuilder().build(inputStream);
```
## SqlSession

### 什么是SqlSession？
SqlSession是MyBatis的关键对象，是持久化操作的独享，类似于jdbc中的Connection，它是应用程序与持久层之间执行交互操作的一个单线程对象。

SqlSession对象包含全部的以数据库为背景的SQL操作方法，底层封装jdbc连接，可以用SqlSession实例来直接执行被映射的SQL语句。

但是记住，**SqlSession应该是线程私有的，因为它不具备线程安全性**。

### SqlSession的获取？

SqlSession的实例需要从SqlSessionFactory中获取。

SqlSession涵盖了对数据库执行SQL命令所需的全部方法。

```java
SqlSession session = sqlSessionFactory.openSession();// 通过SqlSessionFactory对象来获取
try {
  Blog blog = (Blog) session.selectOne("org.mybatis.example.BlogMapper.selectBlog", 101);
} finally {
  session.close();
}
```
上面这种代码比较偏旧。现在有一种更直白的方式。

使用对于给定语句能够合理描述参数和返回值的接口，现在不但可以执行更清晰的类型安全的代码，而且还不用担心易错的字符串字面量以及强制类型转换。

例如：
```java
SqlSession session = sqlSessionFactory.openSession();
try {
  BlogMapper mapper = session.getMapper(BlogMapper.class);
  Blog blog = mapper.selectBlog(101);
} finally {
  session.close();
}
```

### 探究已映射的SQL语句

一个查询的SQL，基于xml的映射语句的实例如下：

```yaml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="org.mybatis.example.BlogMapper">
  <select id="selectBlog" resultType="Blog">
    select * from Blog where id = #{id}
  </select>
</mapper>
```
在一个xml映射文件中，定义多少个映射语句都是可以的。

命名空间“org.mybatis.example.BlogMapper”定义了一个名为“selectBlog”的映射语句。

这样它就允许你使用指定的完全限定名“org.mybatis.example.BlogMapper.selectBlog”来调用映射语句。

这种使用全限定名调用映射语句的方式和通过全限定名调用Java对象的方法是相似的，这是因为mybatis通过这种方式可以直接映射到在命名空间中同名的Mapper类（实际上命名空间就是用于定义对应Mapper类的全限定名的），并将已映射的select语句中的名字、参数、返回值类型匹配成方法。

这样就可以轻松的调用到位于Mapper接口中对应的同名方法。

使用如下的代码：

```java
BlogMapper mapper = session.getMapper(BlogMapper.class);
Blog blog = mapper.selectBlog(101);
```

更有优势，首先它不是基于字符串常量的，就会更安全；

其次，对于已经映射的SQL语句，对于id = "selectBlog"，IDE可以自动为我们补全方法或提示。

## 命名空间

命名空间（namespace）在之前版本的mybatis中是可选的，但是在目前版本的mybatis中是必须的。

命名空间使得我们所见的接口绑定成为可能，出于长远考虑，使用命名空间，并将它置于合适的Java包命名空间之下，将拥有一份更加整洁的代码并提高mybatis的可用性。

## 命名解析
为了减少输入量，mybatis对所有的命名配置元素（包括语句，结果映射，缓存等）使用如下的命名解析规则。

1.全限定名（如“com.package.Mapper.selectAllThings”），将被直接查找并且找到即用。

2.短名称（如“selectAllThings”），如果全局唯一也可以作为一个单独的引用。如果不唯一，就会在使用时收到错误报告，提示短名称不是唯一的，这种情况下必须使用完全限定名。

## 基于注解的SQL映射

映射器类（Mapper class 实际上是一个仅仅包含一系列定义方法签名和返回值类型的interface）还有另一种不是用xml来映射SQL语句的方法—Java注解，如下所示：

```java
package org.mybatis.example;
public interface BlogMapper {
  @Select("SELECT * FROM blog WHERE id = #{id}")
  Blog selectBlog(int id);
}
```
但是注解中的SQL语句毕竟限制性太强，稍微复杂一点的SQL语句会使得代码非常混乱，因此我们应当合理权衡注解和xml的SQL映射。