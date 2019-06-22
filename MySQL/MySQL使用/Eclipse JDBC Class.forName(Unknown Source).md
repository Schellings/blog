---
title: Eclipse JDBC Class.forName(Unknown Source)
date: 2019-02-18 16:56:24
tags: MySQL
---
# 问题描述

在执行Class.forName(driver);时，控制台报出以下错误：

```
java.sql.DriverManager.getConnection(Unknown Source)
Class.forName(Unknown Source)
```

# 问题解决

1、可能是jar包没有导入成功，之前是右键工程->properties->java build path->libraries->classpath->add external jars 添加后项目目录也出现了references jar。



2、可是这样还不行，因为在部署Tomcat的时候没有这个包，解决方法如下：
右键工程->properties->Deployment Assembly->Add->apply and close
再运行，就成功了。

