---
title: String与Long源码解析
date: 2019-08-20 19:56:24
tags: 
    - 源码
    - Java
---

## 前言
String 和 Long 大家都很熟悉，本小节主要结合实际的工作场景，来一起看下 String 和 Long
的底层源码实现，看看平时我们使用时，有无需要注意的点，总结一下这些 API 都适用于哪些
场景。

## String
### 不变性
我们常常听人说，HashMap 的 key 建议使用不可变类，比如说 String 这种不可变类。这里说
的不可变指的是类值一旦被初始化，就不能再被改变了，如果被修改，将会是新的类，我们写个
demo 来演示一下。

```java
String s="Hello";
s="World";
```
从代码上来看，s 的值好像被修改了，但从 debug 的日志来看，其实是 s 的内存地址已经被修
改了，也就说 s =“world” 这个看似简单的赋值，其实已经把 s 的引用指向了新的 String，
debug 的截图显示内存地址已经被修改，两张截图如下：

![](https://img.mukewang.com/5d5fc04a0001c6a508840096.png)

![](https://img.mukewang.com/5d5fc06400019cc210540090.png)

我们从源码上查看一下原因：

```java
public final class String
implements java.io.Serializable, Comparable<String>, CharSequence {
    /** The value is used for character storage. */
    private final char value[];
}
```

1. String 被 final 修饰，说明 String 类绝不可能被继承了，也就是说任何对 String 的操作方
法，都不会被继承覆写；

2. String 中保存数据的是一个 char 的数组 value。我们发现 value 也是被 final 修饰的，也就
是说 value 一旦被赋值，内存地址是绝对无法修改的，而且 value 的权限是 private 的，外
部绝对访问不到，String 也没有开放出可以对 value 进行赋值的方法，所以说 value 一旦产
生，内存地址就根本无法被修改。

以上两点就是 String 不变性的原因，充分利用了 final 关键字的特性，如果你自定义类时，希
望也是不可变的，也可以模仿 String 的这两点操作。

因为 String 具有不变性，所以 String 的大多数操作方法，都会返回新的 String，如下面这种写
法是不对的：

```java
String str ="hello world !!";
// 这种写法是替换不掉的，必须接受 replace 方法返回的参数才行，这样才行：str = str.replace("l",
str.replace("l","dd");

```

### 字符串乱码
在工作中，我们经常碰到这样的场景，进行二进制转化操作时，本地测试的都没有问题，到其它
环境机器上时，有时会出现字符串乱码的情况，这个主要是因为在二进制转化操作时，并没有强
制规定文件编码，而不同的环境默认的文件编码不一致导致的。

```java
String str ="nihao 你好";
// 字符串转化成 byte 数组
byte[] bytes = str.getBytes("ISO-8859-1");
// byte 数组转化成字符串
String s2 = new String(bytes);
log.info(s2);
// 结果打印为：
nihao ??

```

打印的结果为 nihao ?? ，这就是常见的乱码表现形式。这时候有人说，是不是我把代码修改成 Stri
ng s2 = new String(bytes,"ISO-8859-1"); 就可以了？

这是不行的。主要是因为 ISO-8859-1
这种编码对中文的支持有限，导致中文会显示乱码。唯一的解决办法，就是在所有需要用到编码
的地方，都统一使用 UTF-8，对于 String 来说，getBytes 和 new String 两个方法都会使用
到编码，我们把这两处的编码替换成 UTF-8 后，打印出的结果就正常了。





