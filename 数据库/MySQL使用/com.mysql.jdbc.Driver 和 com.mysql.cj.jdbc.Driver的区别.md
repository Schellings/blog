---
title: com.mysql.jdbc.Driver 和 com.mysql.cj.jdbc.Driver的区别
date: 2018-02-14 16:56:24
tags: MySQL
---
一句话：

- com.mysql.jdbc.Driver 是 mysql-connector-java 5中的，

- com.mysql.cj.jdbc.Driver 是 mysql-connector-java 6中的。

所以在使用jdbc时的driver_uri的取值，取决于项目中依赖的mysql-connector-java版本。

其中：

1、JDBC连接Mysql com.mysql.jdbc.Driver 5:

```
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
```
2、JDBC连接Mysql com.mysql.cj.jdbc.Driver 6， 需要指定时区serverTimezone:

```
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
```
在设定时区的时候，如果设定serverTimezone=UTC，会比中国时间早8个小时，如果在中国，可以选择Asia/Shanghai或者Asia/Hongkong，例如：

```
driverClassName=com.mysql.cj.jdbc.Driver
url=jdbc:mysql://localhost:3306/test?serverTimezone=Shanghai&useUnicode=true&characterEncoding=utf8&useSSL=false
username=root
password=
```
# 备注：

I、如果mysql-connector-java用的6.0以上的，如下：

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>6.0.6</version>
</dependency>
```
但是你的driver用的还是com.mysql.jdbc.Driver，就会报错：

```
Loading class 'com.mysql.jdbc.Driver'. This is deprecated. The new 
driver class is 'com.mysql.cj.jdbc.Driver'. 
The driver is automatically registered via the SPI 
and manual loading of the driver class is generally unnecessary.
```

此时需要把com.mysql.jdbc.Driver 改为com.mysql.cj.jdbc.Driver

II、还有一个警告：

```
WARN: Establishing SSL connection without server’s identity verification is not recommended. 
According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection 
must be established by default if explicit option isn’t set. 
For compliance with existing applications not using SSL the verifyServerCertificate property is set to ‘false’. 
You need either to explicitly disable SSL by setting useSSL=false, 
or set useSSL=true and provide truststore for server certificate verification.
```

不推荐不使用服务器身份验证来建立SSL连接。
如果未明确设置，MySQL 5.5.45+, 5.6.26+ and 5.7.6+版本默认要求建立SSL连接。
为了符合当前不使用SSL连接的应用程序，verifyServerCertificate属性设置为’false’。

如果你不需要使用SSL连接，你需要通过设置useSSL=false来显式禁用SSL连接。
如果你需要用SSL连接，就要为服务器证书验证提供信任库，并设置useSSL=true。

SSL – Secure Sockets Layer（安全套接层）
