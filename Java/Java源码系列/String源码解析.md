---
title: String源码解析
date: 2018-08-20 19:56:24
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

### 首字母大小写

如果我们的项目被 Spring 托管的话，有时候我们会通过 **applicationContext.getBean(className);** 这种方式得到 SpringBean，这时
className 必须是要满足首字母小写的，除了该场景，在反射场景下面，我们也经常要使类属性的首字母小写，这时候我们一般都会这么做：

**name.subString(0,1).toLowerCase()+name.subString(1)**,使用 substring 方法，该方法主要是为了截取字符串连续的一部分，substring 有两个方法：

1、public String substring(int beginIndex, int endIndex) beginIndex：开始位置，endIndex：结束位置；

2、public String substring(int beginIndex)beginIndex：开始位置，结束位置为文本末尾。

substring 方法的底层使用的是字符数组范围截取的方法 ： Arrays.copyOfRange(字符数组, 开始
位置, 结束位置); 从字符数组中进行一段范围的拷贝。

相反的，如果要修改成首字母大写，只需要修改成 **name.substring(0, 1).toUpperCase() + nam
e.substring(1)** 即可。

### 相等判断

我们判断相等有两种办法，equals 和 equalsIgnoreCase。后者判断相等时，会忽略大小写，
近期看见一些面试题在问：如果让你写判断两个 String 相等的逻辑，应该如何写，我们来一起
看下 equals 的源码，整理一下思路：

```java
public boolean equals(Object anObject) {
    // 判断内存地址是否相同
    if (this == anObject) {
        return true;
    }
    // 待比较的对象是否是 String，如果不是 String，直接返回不相等
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = value.length;
        // 两个字符串的长度是否相等，不等则直接返回不相等
        if (n == anotherString.value.length) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            // 依次比较每个字符是否相等，若有一个不等，直接返回不相等
            while (n-- != 0) {
                if (v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```
从 equals 的源码可以看出，逻辑非常清晰，完全是根据 String 底层的结构来编写出相等的代
码。这也提供了一种思路给我们：如果有人问如何判断两者是否相等时，我们可以从两者的底层
结构出发，这样可以迅速想到一种贴合实际的思路和方法，就像 String 底层的数据结构是 char
的数组一样，判断相等时，就挨个比较 char 数组中的字符是否相等即可。


### 替换、删除

替换在工作中也经常使用，有 replace 替换所有字符、replaceAll 批量替换字符串、
replaceFirst 替换遇到的第一个字符串三种场景。
其中在使用 replace 时需要注意，replace 有两个方法，一个入参是 char，一个入参是
String，前者表示替换所有字符，如： name.replace('a','b') ，后者表示替换所有字符串，如：
name.replace("a","b")，两者就是单引号和多引号的区别。

需要注意的是， replace 并不只是替换一个，是替换所有匹配到的字符或字符串哦。

写了一个 demo 演示一下三种场景：

```java
public void testReplace(){
  String str ="hello word !!";
  log.info("替换之前 :{}",str);
  str = str.replace('l','d');
  log.info("替换所有字符 :{}",str);
  str = str.replaceAll("d","l");
  log.info("替换全部 :{}",str);
  str = str.replaceFirst("l","");
  log.info("替换第一个 l :{}",str);
}
//输出的结果是：
替换之前 :hello word !!
替换所有字符 :heddo word !!
替换全部 :hello worl !!
替换第一个 :helo worl !!
```
当然我们想要删除某些字符，也可以使用 replace 方法，把想删除的字符替换成 "" 即可。

### 拆分和合并

拆分我们使用 split 方法，该方法有两个入参数。第一个参数是我们拆分的标准字符，第二个参
数是一个 int 值，叫 limit，来限制我们需要拆分成几个元素。如果 limit 比实际能拆分的个数小，按照limit的个数拆分。

```java
String s ="boo:and:foo";
// 我们对 s 进行了各种拆分，演示的代码和结果是：
s.split(":") 结果:["boo","and","foo"]
s.split(":",2) 结果:["boo","and:foo"]
s.split(":",5) 结果:["boo","and","foo"]
s.split(":",-2) 结果:["boo","and","foo"]
s.split("o") 结果:["b","",":and:f"]
s.split("o",2) 结果:["b","o:and:foo"]
```
从演示的结果来看，limit 对拆分的结果，是具有限制作用的，还有就是拆分结果里面不会出现
被拆分的字段。
那如果字符串里面有一些空值呢，拆分的结果如下：

```java
String a =",a,,b,";
a.split(",") 结果:["","a","","b"]
```

从拆分结果中，我们可以看到，空值是拆分不掉的，仍然成为结果数组的一员，如果我们想删除
空值，只能自己拿到结果后再做操作，但 Guava（Google 开源的技术工具） 提供了一些可靠
的工具类，可以帮助我们快速去掉空值，如下：

```java
String a =",a, ,  b  c ,";
// Splitter 是 Guava 提供的 API 
List<String> list = Splitter.on(',')
    .trimResults()// 去掉空格
    .omitEmptyStrings()// 去掉空值
    .splitToList(a);
log.info("Guava 去掉空格的分割方法：{}",JSON.toJSONString(list));
// 打印出的结果为：
["a","b  c"]
```
从打印的结果中，可以看到去掉了空格和空值，这正是我们工作中常常期望的结果，所以推荐使
用 Guava 的 API 对字符串进行分割。


合并我们使用 join 方法，此方法是静态的，我们可以直接使用。方法有两个入参，参数一是合
并的分隔符，参数二是合并的数据源，数据源支持数组和 List，在使用的时候，我们发现有两个
不太方便的地方：

1. 不支持依次 join 多个字符串，比如我们想依次 join 字符串 s 和 s1，如果你这么写的话 Stri
ng.join(",",s).join(",",s1) 最后得到的是 s1 的值，第一次 join 的值被第二次 join 覆盖了；

2. 如果 join 的是一个 List，无法自动过滤掉 null 值。

而 Guava 正好提供了 API，解决上述问题，我们来演示一下：

```java
// 依次 join 多个字符串，Joiner 是 Guava 提供的 API
Joiner joiner = Joiner.on(",").skipNulls();
String result = joiner.join("hello",null,"china");
log.info("依次 join 多个字符串:{}",result);

List<String> list = Lists.newArrayList(new String[]{"hello","china",null});
log.info("自动删除 list 中空值:{}",joiner.join(list));
// 输出的结果为；
依次 join 多个字符串:hello,china
自动删除 list 中空值:hello,china
```

从结果中，我们可以看到 Guava 不仅仅支持多个字符串的合并，还帮助我们去掉了 List 中的
空值，这就是我们在工作中常常需要得到的结果。






