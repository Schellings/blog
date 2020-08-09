---
title: Java Class反编译
date: 2020-03-31 22:11:12
tags: 
    - Java
---
<meta name="referrer" content="no-referrer" />

## 什么是反编译?

```yaml
反编译的过程与编译刚好相反，就是将已编译好的编程语言还原到未编译的状态，也就是找出程序语言的源代码。就是将机器看得懂的语言转换成程序员可以看得懂的语言。Java语言中的反编译一般指将class文件转换成java文件。

有了反编译工具，我们可以做很多事情，最主要的功能就是有了反编译工具，我们就能读得懂Java编译器生成的字节码。比如我们就可以洞悉Java语法糖背后的原理。
```

## Java常用反编译工具

本文主要介绍4个Java的反编译工具：javap、jad和cfr以及可视化反编译工具JD-GUI

### javap

```yaml
javap是jdk自带的一个工具，可以对代码反编译，也可以查看java编译器生成的字节码。javap和其他两个反编译工具最大的区别是他生成的文件并不是java文件，也不像其他两个工具生成代码那样更容易理解。
```

拿一段简单的代码举例，如我们想分析Java 7中的switch是如何支持String的，我们先有以下可以编译通过的源代码：

```java
public class switchDemoString {
    public static void main(String[] args) {
        String str = "world";
        switch (str) {
            case "hello":
                System.out.println("hello");
                break;
            case "world":
                System.out.println("world");
                break;
            default:
                break;
        }
    }
}
```

执行以下两个命令：

`javac Decompilation.java`
`javap -c Decompilation.class`

得到以下信息：
```java
Compiled from "Decompilation.java"
public class Decompilation {
  public Decompilation();
    Code:
       0: aload_0
       1: invokespecial #8                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: ldc           #16                 // String world
       2: astore_1
       3: aload_1
       4: dup
       5: astore_2
       6: invokevirtual #18                 // Method java/lang/String.hashCode:()I
       9: lookupswitch  { // 2
              99162322: 36
             113318802: 48
               default: 82
          }
      36: aload_2
      37: ldc           #24                 // String hello
      39: invokevirtual #26                 // Method java/lang/String.equals:(Ljava/lang/Object;)Z
      42: ifne          60
      45: goto          82
      48: aload_2
      49: ldc           #16                 // String world
      51: invokevirtual #26                 // Method java/lang/String.equals:(Ljava/lang/Object;)Z
      54: ifne          71
      57: goto          82
      60: getstatic     #30                 // Field java/lang/System.out:Ljava/io/PrintStream;
      63: ldc           #24                 // String hello
      65: invokevirtual #36                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      68: goto          82
      71: getstatic     #30                 // Field java/lang/System.out:Ljava/io/PrintStream;
      74: ldc           #16                 // String world
      76: invokevirtual #36                 // Method java/io/PrintStream.println:(Ljava/lang/String;)V
      79: goto          82
      82: return
}
```

javap并没有将字节码反编译成java文件，而是生成了一种我们可以看得懂字节码。其实javap生成的文件仍然是字节码，只是程序员可以稍微看得懂一些。如果你对字节码有所掌握，还是可以看得懂以上的代码的。其实就是把String转成hashcode，然后进行比较。

### jad

JAD是一个比较不错的反编译工具，只要下载一个执行工具，就可以实现对class文件的反编译了。还是上面的源代码，使用jad反编译后内容如下：

命令：`jad.exe Decompilation.class` 会生成一个Decompilation.jad的文件

```java
// Decompiled by Jad v1.5.8g. Copyright 2001 Pavel Kouznetsov.
// Jad home page: http://www.kpdus.com/jad.html
// Decompiler options: packimports(3) 
// Source File Name:   Decompilation.java

package com.yveshe;

import java.io.PrintStream;

public class Decompilation
{

    public Decompilation()
    {
    }

    public static void main(String args[])
    {
        String str = "world";
        String s;
        switch((s = str).hashCode())
        {
        default:
            break;

        case 99162322: 
            if(s.equals("hello"))
                System.out.println("hello");
            break;

        case 113318802: 
            if(s.equals("world"))
                System.out.println("world");
            break;
        }
    }
}
```

看上面的代码这不就是标准的java的源代码么。这个就很清楚的可以看到原来字符串的switch是通过equals()和hashCode()方法来实现的。

`PS: 但是，由于JAD已经很久不更新了，在对Java7生成的字节码进行反编译时，偶尔会出现不支持的问题，在对Java 8的lambda表达式反编译时就彻底失败。`

### cfr
```yaml
JAD很好用，但是无奈的是很久没更新了，所以只能用一款新的工具替代他，CFR是一个不错的选择，相比JAD来说，他的语法可能会稍微复杂一些，但是好在他可以用.

CFR将反编译现代Java特性–Java 8 lambdas（Java和更早版本中的Java beta 103），已经反编译Java 7 String，但CFR是完全用Java 6编写的.
```
我们使用CFR对刚刚的代码进行反编译。执行一下命令：
`java -jar cfr_0_125.jar Decompilation.class --decodestringswitch false`

得到以下信息：

```java
/*
 * Decompiled with CFR 0_125.
 */
package com.yveshe;

import java.io.PrintStream;

public class Decompilation {
    public static void main(String[] args) {
        String str;
        String s = str = "world";
        switch (s.hashCode()) {
            default: {
                break;
            }
            case 99162322: {
                if (!s.equals("hello")) break;
                System.out.println("hello");
                break;
            }
            case 113318802: {
                if (!s.equals("world")) break;
                System.out.println("world");
            }
        }
    }
}
```

### JD-GUI

JD-GUI 是一个用 C++ 开发的 Java反编译工具，由 Pavel Kouznetsov开发，支持Windows、Linux和苹果Mac Os三个平台。而且提供了Eclipse平台下的插件JD-Eclipse。JD-GUI 基于GPLv3开源协议，对个人使用是完全免费的。JD-GUI主要的是提供了可视化操作,直接拖拽文件到窗口既可,效果图如下

![](https://img-blog.csdn.net/20180420173831147?watermark/2/text/aHR0cDovL3d3dy55dmVzaGUuY29t/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 如何防止反编译?

由于我们有工具可以对Class文件进行反编译，所以，对开发人员来说，如何保护Java程序就变成了一个非常重要的挑战。但是，魔高一尺、道高一丈。当然有对应的技术可以应对反编译咯。但是，这里还是要说明一点，和网络安全的防护一样，无论做出多少努力，其实都只是提高攻击者的成本而已。无法彻底防治。

典型的应对策略有以下几种：
- 隔离Java程序
- 让用户接触不到你的Class文件
- 对Class文件进行加密
- 提到破解难度
- 代码混淆
- 将代码转换成功能上等价，但是难于阅读和理解的形式、比如变量全部替换__、____的形式

