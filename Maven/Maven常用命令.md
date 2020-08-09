---
title: Maven常用命令
date: 2018-02-25 16:56:24
tags: Maven
---
<meta name="referrer" content="no-referrer" />
# Maven常用命令：
## 1. 创建Maven的普通Java项目：


```
mvn archetype:create
    -DgroupId=packageName
    -DartifactId=projectName
```

## 2. 创建Maven的Web项目：


```
mvn archetype:create
    -DgroupId=packageName
    -DartifactId=webappName
    -DarchetypeArtifactId=maven-archetype-webapp
```

## 3. 反向生成 maven 项目的骨架：


```
mvn archetype:generate
```

你是怎么创建你的maven项目的?是不是像这样:


```
mvn archetype:create -DarchetypeArtifactId=maven-archetype-quickstart -DgroupId=com.ryanote -Dartifact=common
```

如果你还再用的话,那你就out了,现代人都用

**mvn archetype:generate**

它将创建项目这件枯燥的事更加人性化,你再也不需要记那么多的archetypeArtifactId,你只需输入archetype:generate,剩下的就是做”选择题”了。
　　
![image](https://images2015.cnblogs.com/blog/983815/201701/983815-20170127032346612-1566232122.png)

缩写写法：

```
mvn archetype:generate -DgroupId=otowa.user.dao -DartifactId=user-dao -Dversion=0.01-SNAPSHOT
```

## 4. 编译源代码：


```
mvn compile
```

## 5. 编译测试代码：


```
mvn test-compile
```

## 6. 运行测试:


```
mvn test
```

## 7. 产生site：


```
mvn site
```

## 8. 打包：


```
mvn package
```

## 9. 在本地Repository中安装jar：


```
mvn install
```


## 10. 清除产生的项目：


```
mvn clean
```
## 11. 生成eclipse项目：


```
mvn eclipse:eclipse
```

## 12. 生成idea项目：


```
mvn idea:idea
```

## 13. 组合使用goal命令，如只打包不测试：


```
mvn -Dtest package
```


## 14. 只打jar包:


```
mvn jar:jar
```

## 15. 只测试而不编译，也不测试编译：


```
mvn test -skipping compile -skipping test-compile
```


## 16. 清除eclipse的一些系统设置:


```
mvn eclipse:clean
```
 
## 17.查看当前项目已被解析的依赖：


```
mvn dependency:list
```

## 18.上传到私服：


```
mvn deploy
```

## 19. 强制检查更新。
由于快照版本的更新策略(一天更新几次、隔段时间更新一次)存在，如果想强制更新就会用到此命令: 


```
mvn clean install-U
```

## 20. 源码打包：


```
mvn source:jar
或
mvn source:jar-no-fork
```

## 21、项目运行
需要在pom文件中添加相应的plugin运行插件

运行项目于jetty上:

```
mvn jetty:run
```
运行项目于tomcat上:

```
mvn tomcat:run
```

运行时的常用参数：


```
1. 跳过测试:-Dmaven.test.skip=true

2. 指定端口:-Dmaven.tomcat.port=9090

3. 忽略测试失败:-Dmaven.test.failure.ignore=true
```


各个参数之间用空格分隔。

# mvn compile与mvn install、mvn deploy的区别
1、mvn compile，编译类文件

2、mvn install，包含mvn compile，mvn package，然后上传到本地仓库

3、mvn deploy,包含mvn install,然后，上传到私服

