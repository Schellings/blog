---
title: Java8流（stream）操作
date: 2020-03-28 22:11:12
tags: 
    - Java
---
<meta name="referrer" content="no-referrer" />

## Stream是什么

```yaml
Java 8 中的 Stream 是对集合（Collection）对象功能的增强，它专注于对集合对象进行各种非常便利、高效的聚合操作，或者大批量数据操作 。
Stream API 借助于同样新出现的 Lambda 表达式，极大的提高编程效率和程序可读性.
```

以前我们处理复杂的数据只能通过各种for循环，不仅不美观，而且时间长了以后可能自己都看不太明白以前的代码了，但有Stream以后，通过filter，map，limit等等方法就可以使代码更加简洁并且更加语义化。

```yaml
Stream可以先把数据变成符合要求的样子（map），吃掉不需要的东西（filter）然后得到需要的东西（collect）。
```

## Stream的使用

```java
List<Dish> menu = ...

List<String> lowCaloricDishesName = menu.stream()
    //筛选出卡路里大于400的
    .filter(d -> d.getCalories() < 400)
    //抽取名字属性创建一个新的流
    .map(Dish::getName)
    //这个流按List类型返回
    .collect(toList());
```

在这段代码 filter 和 map 操作被称为中间操作,中间操作会返回一个新的流，而 collect 则被称为终端操作只有终端操作才会让整个流执行并关闭。也就是说 `每个流只能遍历一次` ，因为collect以后这个流就已经关闭了。

```java
List<String> test = Arrays.asList("Java8", "In", "Action");
Stream<String> s = title.stream();
s.forEach(System.out::println);
s.forEach(System.out::println);   // 代码会抛出一个java.lang.IllegalStateException异常
```

想了解更多Stream的api可以查阅[官方文档](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)。

## 串行与并行

Stream可以分为串行与并行两种，串行流和并行流差别就是单线程和多线程的执行。

- default Stream stream() ： 返回串行流
- default Stream parallelStream() ： 返回并行流

stream()和parallelStream()方法返回的都是java.util.stream.Stream<E>类型的对象，说明它们在功能的使用上是没差别的。唯一的差别就是单线程和多线程的执行。

## 性能问题

```yaml
1.对于简单操作，比如最简单的遍历，Stream串行API性能明显差于显示迭代，但并行的Stream API能够发挥多核特性。
2.对于复杂操作，Stream串行API性能可以和手动实现的效果匹敌，在并行执行时Stream API效果远超手动实现。
所以，如果出于性能考虑，1. 对于简单操作推荐使用外部迭代手动实现，2. 对于复杂操作，推荐使用Stream API， 3. 在多核情况下，推荐使用并行Stream API来发挥多核优势，4.单核情况下不建议使用并行Stream API。

如果出于代码简洁性考虑，使用Stream API能够写出更短的代码。即使是从性能方面说，尽可能的使用Stream API也另外一个优势，那就是只要Java Stream类库做了升级优化，代码不用做任何修改就能享受到升级带来的好处。
```
