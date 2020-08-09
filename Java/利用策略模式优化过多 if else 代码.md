---
title: 利用策略模式优化过多 if else 代码
date: 2020-04-05 22:11:12
tags: 
    - Java
---
<meta name="referrer" content="no-referrer" />

## 背景

比如平时大家是否都会写类似这样的代码：
```java
if(a){
    //dosomething
}else if(b){
    //doshomething
}else if(c){
    //doshomething
} else{
    ////doshomething
}
```

条件少还好，一旦 `else if` 过多这里的逻辑将会比较混乱，并很容易出错。
比如：
![](https://upload-images.jianshu.io/upload_images/2038379-09d7381ceaf2dac5.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
刚开始条件较少，也就没管那么多直接写的；现在功能多了导致每次新增一个 else 条件我都得仔细核对，生怕影响之前的逻辑。

这次终于忍无可忍就把他重构了，重构之后这里的结构如下：

![](https://upload-images.jianshu.io/upload_images/2038379-35ed8628bbd252b6.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

最后直接变为两行代码，简洁了许多。

而之前所有的实现逻辑都单独抽取到其他实现类中。

![](https://upload-images.jianshu.io/upload_images/2038379-413f1b2746924cfb.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/496/format/webp)

![](https://upload-images.jianshu.io/upload_images/2038379-4d231e6595d8409c.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
这样每当我需要新增一个 else 逻辑，只需要新增一个类实现同一个接口便可完成。每个处理逻辑都互相独立互不干扰。

## 实现

![](https://upload-images.jianshu.io/upload_images/2038379-ac9cf7d5157e2ad4.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)



