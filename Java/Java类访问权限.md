---
title: Java类访问权限
date: 2017-05-15 19:56:24
tags: 
    - Java
---

java的访问权限有四种，public，protected，默认（friendly），private

四种权限作用图。Y代表可以访问到。

![](https://img-blog.csdn.net/20160503100451023?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

类的成员方法的访问权限： 
其从语法角度来讲，这写访问权限控制符作用于类的方法和作用于类的属性上的效果是一样的。 

- public：所有类可见。 
- private：只有同一类内部的方法可见，在有就是内部类也可以访问到。 
- 默认（friendly）：包内可见。 
- protected:继承可见。 