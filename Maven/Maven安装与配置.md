---
title: Maven安装与配置
date: 2018-02-23 16:56:24
tags: Maven
---
# 1、maven初识
Maven 翻译为"专家"、"内行"，是 Apache 下的一个纯 Java 开发的开源项目。基于项目对象模型（缩写：POM）概念，Maven利用一个中央信息片断能管理一个项目的构建、报告和文档等步骤。

Maven 是一个项目管理工具，可以对 Java 项目进行构建、依赖管理。

Maven 也可被用于构建和管理各种项目，例如 C#，Ruby，Scala 和其他语言编写的项目。Maven 曾是 Jakarta 项目的子项目，现为由 Apache 软件基金会主持的独立 Apache 项目。
# 2、约定配置
Maven 提倡使用一个共同的标准目录结构，Maven 使用约定优于配置的原则，大家尽可能的遵守这样的目录结构。如下所示：

![image](http://chatbot.xielin.top/maven/maven01.jpg)

# 3、Maven 特点
1、项目设置遵循统一的规则。

2、任意工程中共享。

3、依赖管理包括自动更新。

4、一个庞大且不断增长的库。

5、可扩展，能够轻松编写 Java 或脚本语言的插件。

6、只需很少或不需要额外配置即可即时访问新功能。

7、**基于模型的构建** − Maven能够将任意数量的项目构建到预定义的输出类型中，如 JAR，WAR 或基于项目元数据的分发，而不需要在大多数情况下执行任何脚本。

8、**项目信息的一致性站点** − 使用与构建过程相同的元数据，Maven 能够生成一个网站或PDF，包括您要添加的任何文档，并添加到关于项目开发状态的标准报告中。

9、**发布管理和发布单独的输出** − Maven 将不需要额外的配置，就可以与源代码管理系统（如 Subversion 或 Git）集成，并可以基于某个标签管理项目的发布。它也可以将其发布到分发位置供其他项目使用。Maven 能够发布单独的输出，如 JAR，包含其他依赖和文档的归档，或者作为源代码发布。

10、**向后兼容性** − 您可以很轻松的从旧版本 Maven 的多个模块移植到 Maven 3 中。

**子项目使用父项目依赖时，正常情况子项目应该继承父项目依赖，无需使用版本号**。

**并行构建** − 编译的速度能普遍提高20 - 50 %。

**更好的错误报告** − Maven 改进了错误报告，它为您提供了 Maven wiki 页面的链接，您可以点击链接查看错误的完整描述。

# 4、maven的安装
1、Maven 是一个基于 Java 的工具，所以要做的第一件事情就是安装 JDK，并配置好环境变量JAVA_HOME。

2、下载解压Maven包。

进入Maven 下载地址：http://maven.apache.org/download.cgi 

![image](https://www.runoob.com/wp-content/uploads/2018/09/750D721E-0624-4C16-AD4B-9EA5D7F6289A.png)
选择相应平台，下载后，解压出来。

3、配置环境变量

将maven的解压地址，配置为环境变量MAVEN_HOME。即

key:MAVEN_HOME

value:E:\Maven\apache-maven-3.3.9(根据maven解压地址变换)

将%MAVEN_HOME%\bin添加到系统path中；

4、开启cmd，输入命令mvn -v，如果正确输出maven版本信息，则maven安装成功。

# 5、maven的常用配置
maven的配置文件，是在maven解压文件夹/conf/settings.xml文件中进行相关配置。
## 5.1、阿里云maven仓库配置

修改settings.xml文件，找到mirrors节点，将其替换为以下内容：

```
<mirrors>
   <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>       
	</mirror>
  </mirrors>
```

## 5.2、修改本地maven仓库地址

修改settings.xml文件，找到localRepository节点，本节点的配置默认是被注释掉的，关闭该注释，将其替换为以下内容：

```
<localRepository><localRepository>D:\WorkSpace\apache-maven-3.6.1\mvnRepository</localRepository>（自定义的本地仓库地址）</localRepository>
```
