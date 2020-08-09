---
title: override注解报错，接口无法实现
date: 2020-02-11 19:56:24
tags: 
    - Java
---
<meta name="referrer" content="no-referrer" />
## @override 注解报错

在项目依赖中增加以下配置：

```
   <build>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-compiler-plugin</artifactId>
               <version>2.3.2</version>
               <configuration>
                   <source>1.8</source>
                   <target>1.8</target>
               </configuration>
           </plugin>
       </plugins>
   </build>
```

或者通过操作：

idea: 
![](https://img-blog.csdnimg.cn/20190114170524639.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQ2NDk2NA==,size_16,color_FFFFFF,t_70)

eclipse:

![](https://img-blog.csdn.net/20180116151906299?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdTAxMTkwMDQ0OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
