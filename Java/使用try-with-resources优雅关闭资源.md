---
title: 使用try-with-resources优雅关闭资源
date: 2020-03-12 22:11:12
tags: 
    - Java
---
<meta name="referrer" content="no-referrer" />

## 用法辨析

Java库中有很多资源需要手动关闭，比如InputStream、OutputStream、java.sql.Connection等等。在此之前，通常是使用try-finally的方式关闭资源；Java7之后，推出了try-with-resources声明来替代之前的方式。 

try-with-resources 声明要求其中定义的变量实现 AutoCloseable 接口，这样系统可以自动调用它们的close方法，从而替代了finally中关闭资源的功能。

举个栗子，下面是一个复制文件的方法，按照原本`try-catch-finally`的写法：
```java
// 一个简单的复制文件方法。
public static void copy(String src, String dst) {
    InputStream in = null;
    OutputStream out = null;
    try {
        in = new FileInputStream(src);
        out = new FileOutputStream(dst);
        byte[] buff = new byte[1024];
        int n;
        while ((n = in.read(buff)) >= 0) {
            out.write(buff, 0, n);
        }
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (in != null) {
            try {
                in.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        if (out != null) {
            try {
                out.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
下面来看使用了`try-with-resources`后的效果：

```java
public static void copy(String src, String dst) {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buff = new byte[1024];
        int n;
        while ((n = in.read(buff)) >= 0) {
            out.write(buff, 0, n);
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
`try-with-resources`将会自动关闭try()中的资源，并且将先关闭后声明的资源。

如果不catch IOException就更加清爽了：

```java
public static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
         OutputStream out = new FileOutputStream(dst)) {
        byte[] buff = new byte[1024];
        int n;
        while ((n = in.read(buff)) >= 0) {
            out.write(buff, 0, n);
        }
    }
}
```

## 原理分析

那么try-with-resources有什么神奇之处呢？到底做了什么呢？

我们先来看下AutoCloseable接口：

```java
public interface AutoCloseable {
    void close() throws Exception;
}
```

其中仅有一个close方法，实现AutoCloseable接口的类将在close方法中实现其关闭资源的功能。

而`try-with-resources`其实是个语法糖，它将在编译时编译成关闭资源的代码。我们将上述例子中的代码编译成class文件，再反编译回java文件，就能看到如下代码：

```java
public static void copy(String var0, String var1) throws IOException {
    FileInputStream var2 = new FileInputStream(var0);
    Throwable var3 = null;

    try {
        FileOutputStream var4 = new FileOutputStream(var1);
        Throwable var5 = null;

        try {
            byte[] var6 = new byte[1024];

            int var7;
            while((var7 = var2.read(var6)) >= 0) {
                var4.write(var6, 0, var7);
            }
        } catch (Throwable var29) {
            var5 = var29;
            throw var29;
        } finally {
            if (var4 != null) {
                if (var5 != null) {
                    try {
                        // 关闭FileOutputStream
                        var4.close();
                    } catch (Throwable var28) {
                        var5.addSuppressed(var28);
                    }
                } else {
                    var4.close();
                }
            }

        }
    } catch (Throwable var31) {
        var3 = var31;
        throw var31;
    } finally {
        if (var2 != null) {
            if (var3 != null) {
                try {
                    // 关闭FileInputStream
                    var2.close();
                } catch (Throwable var27) {
                    var3.addSuppressed(var27);
                }
            } else {
                var2.close();
            }
        }

    }
}
```

所以，推荐使用try-with-resources机制，但是`能够使用try-with-resources关闭资源的类，必须实现AutoCloseable接口`。