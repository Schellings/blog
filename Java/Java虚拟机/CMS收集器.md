---
title:  CMS收集器
date: 2019-03-18 19:56:24
tags: 
    - Java虚拟机
---
<meta name="referrer" content="no-referrer" />

## 什么是CMS

 CMS(Concurrent Mark Sweep)回收器，它使用的是标记清除算法，同时又是一个使用多线程并行回收的垃圾回收器。
 
 CMS收集器是-种以获取最短回收停顿时间为⽬标的收集器。 ⽬前很⼤⼀部分的Java应⽤集中在互联⽹站或者B/S系统的服务端上,这类应⽤尤其重视服务的响应速度，希望系统停顿时间最短，以给⽤户带来较好的体验。
 
 CMS 收集器是基于“标记-清除”算法实现的，用户线程和垃圾线程同时执行。
 
 用户几乎感受用不到线程的暂停。也就是并发。
 
 ## 处理流程

![](http://huhanlin.com/ueditor/php/upload/image/20161223/1482467323727604.png)

根据标记清除算法，初始标记、并发标记和重新标记都是为了标记出需要回收的对象。并发清理则是在标记完成后，正是回收垃圾对象。并发重置是指在垃圾回收完成后，重新初始化CMS数据结构和数据，为下一次垃圾回收做好准备。

在整个CMS回收过程中，默认情况下，在并发标记之后，会有一个预清理的操作（也可以设置参数-XX:CMSPrecleaningEnabled，不进行预清理）。

预清理是并发的，除了为正式清理做准备和检查以外，预清理还会尝试控制一次停顿时间。

由于重新标记是独占CPU的，如果新生代GC发生后立即触发一次重新标记，那么一次停顿时间可能很长。

为了避免这种情况，预清理时，会可以等待一次新生代GC的发生，然后根据历史性能数据预测一下新生代GC可能发生的时间，然后在当前时间和预测时间的中间时刻，进行重新标记。这样，最大程度上避免新生代GC和重新标记重合，尽可能减少一次停顿时间。

## CMS垃圾收集器缺点

- 对CPU资源⾮常敏感
- ⽆法处理浮动垃圾，程序在进⾏并发清除阶段⽤户线程所产⽣的新垃圾
- 标记清除算法存在空间碎⽚

## CMS主要参数设置

```yaml
-XX:+UseConcMarkSweepGC启用CMS回收器
-XX:ConcGCThreads 设置并发线程数量
-XX:CMSInitiatingOccupancyFraction 设置当老年代使用率达到N时，执行一次CMS回收。
-XX:+UseCMSCompactAtFullCollection CMS在垃圾收集完成后，进行一次碎片整理。
-XX:CMSFullGCsBeforeCompaction 设置当进行N次CMS回收后进行一次内存压缩。
-XX:+CMSClassUnloadingEnabled 使用CMS机制回收Perm区Class数据
```