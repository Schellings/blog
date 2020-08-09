---
title: maven使用之packaging类型
date: 2018-07-21 16:56:24
tags: Maven
---
<meta name="referrer" content="no-referrer" />

项目的打包类型：pom、jar、war、maven-plugin

项目中一般使用maven进行模块管理，每个模块下对应都有一个pom文件，pom文件维护了各个模块之间的依赖和继承关系。项目模块化可以将通用的部分抽离出来，方便重用；修改一部分代码不再是build整个项目，缩短了build时间；此外各模块都有自己的pom文件，结构更清晰。



使用maven进行模块划分管理，一般都会有一个父级项目，pom文件出了GAV(groupId, artifactId, version)是必须要配置的，另一个重要的属性就是packaging打包类型，所有的父级项目的packaging都为pom，packaging默认类型jar类型，如果不做配置，maven会将该项目打成jar包。作为父级项目，还有一个重要的属性，那就是modules，通过modules标签将项目的所有子项目引用进来，在build父级项目时，会根据子模块的相互依赖关系整理一个build顺序，然后依次build。



而对于各个子项目，需要在其对应的pom文件开头声明对父级项目的引用，通过GAV实现。对于子项目中自己的GAV配置，GV如果不配置，则会从父类项目的配置继承过来。子模块可以通过dependencies标签来添加自己的依赖，此外子类项目的packaging值只能是war或jar。如果是需要部署的项目，一般是包含controller的module，需要打包成war类型，如果只是内部调用或者是作服务使用，则推荐打成jar包，是服务于war包的， 位于war包中的lib文件夹下。



maven项目的部署过程：

先创建一个maven的web项目，对该项目打包成war包 ，然后部署到tomcat或者resin上，然后对该项目的class文件打包成jar包，放到项目的lib目录下，这样，项目就可以直接在lib下面的该 jar包下的class文件。