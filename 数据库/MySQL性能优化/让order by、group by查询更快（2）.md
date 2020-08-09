---
title: 让order by、group by查询更快（2）
date: 2018-12-18 16:56:24
tags: 
  - SQL优化
  - MySQL
---
<meta name="referrer" content="no-referrer" />
## 前言

前面我们学习到MySQL中的两种排序方式：index sort + file sort，重点了解了file sort中的两种排序位置：内存 + 文件，其中文件排序包含三种排序方式：单路排序+双路排序+打包数据排序。

下面让我们继续认识一下order by与group by的排序优化方式。

## order by 优化

上面我们分析了 order by 的原理，小伙伴们应该会有些优化 order by 的思路了，下面我们就一起来总结 order by 的一些优化技巧。

### 添加合适索引

#### 排序字段添加索引

首先我们看下对 d 字段（没有索引）进行排序的执行计划：
```yaml
explain select d,id from t1 order by d;
```

![](https://img.mukewang.com/5d3aabce0001a5b013690153.png)

发现使用的是 filesort（关注 Extra 字段）。

再看些对 c 字段（有索引）进行排序的执行计划：

```yaml
explain select c,id from t1 order by c;
```
![](https://img.mukewang.com/5d3aabb30001897113380160.png)

可以看到，根据有索引的字段排序，在 Extra 中显示的就为 Using index，表示使用的是索引排序。如果数据量比较大，显然通过有序索引直接返回有序数据效率更高。因此可以在排序字段上添加索引来优化排序语句。

#### 多个字段排序优化

有时面对的需求是要对多个字段进行排序，而这种情况应该怎么优化或者设计索引呢？首先看下面例子：

对 a、c 两个字段进行排序的执行计划：
```yaml
explain select id,a,c from t1 order by a,c;
```
![](https://img.mukewang.com/5d3aab48000146eb13530156.png)

发现使用的是索引排序。
多个字段排序的情况，如果要通过添加索引优化，得注意**排序字段的顺序与联合索引中列的顺序要一致**。

因此，**如果多个字段排序，可以在多个排序字段上添加联合索引来优化排序语句。**


#### 先等值查询再排序的优化

我们更多的情况是会先根据某个字段条件查出一部分数据，然后再排序，而这类 SQL 应该如果优化呢？看下面的实验：

表 t1中，根据 a=1000 过滤数据再根据 d 字段排序的执行计划如下：

```yaml
explain select id,a,d from t1 where a=1000 order by d;
```
![](https://img.mukewang.com/5d3aab280001b5e715700160.png)

可以在 Extra 字段中看到 “Using filesort”，说明使用的是 filesort 排序。

再看下根据 a=1000 过滤数据在根据 b 字段排序的执行计划（a、b 两个字段有联合索引）：
```yaml
explain select id,a,b from t1 where a=1000 order by b;
```

![](https://img.mukewang.com/5d3aab120001119414780161.png)

可以在 Extra 字段中看到“Using index”，说明使用的是索引排序。

因此，对于先等值查询再排序的语句，可以通过在条件字段和排序字段添加联合索引来优化此类排序语句。

#### 去掉不必要的返回字段

有时，我们其实并不需要查询出所有字段，但是可能因为习惯问题，就写成查所有字段的数据了。我们看下下面两条 SQL 的执行计划：

```yaml
select * from t1 order by a,b; /* 根据a和b字段排序查出所有字段的值 */
select id,a,b from t1 order by a,b; /* 根据a和b字段排序查出id,a,b字段的值 */
```

![](https://img.mukewang.com/5d3aaaf90001e0b812920323.png)

根据执行计划的结果，可以看到，查询所有字段的这条 SQL 是 filesort 排序，而只查 id、a、b 三个字段的 SQL 是 index 排序，为什么查询所有
字段会不走索引？

这个例子中，查询所有字段不走索引的原因是：扫描整个索引并查找到没索引的行的成本比扫描全表的成本更高，所以优化器放弃使用索引。

#### 修改参数

在本节一开始讲 order by 原理的时候，接触到两个跟排序有关的参数：max_length_for_sort_data、sort_buffer_size。

- max_length_for_sort_data：如果觉得排序效率比较低，可以适当加大 max_length_for_sort_data 的值，让优化器优先选择全字段排序。当然
不能设置过大，可能会导致 CPU 利用率过低或者磁盘 I/O 过高；

- sort_buffer_size：适当加大 sort_buffer_size 的值，尽可能让排序在内存中完成。但不能设置过大，可能导致数据库服务器 SWAP。

#### 几种无法利用索引排序的情况

如果要写出高效率的排序 SQL，几种无法利用索引排序的情况应该熟记于心，在写 SQL 是就应该规避掉。

##### 使用范围查询再排序
前面介绍过，对于先等值过滤再排序的语句，可以通过在条件字段和排序字段添加联合索引来优化；但是如果联合索引中前面的字段使
用了范围查询，对后面的字段排序是否能用到索引排序呢？下面我们通过实验验证一下：

```yaml
explain select id,a,b from t1 where a>9000 order by b;
```

![](https://img.mukewang.com/5d3aaae000011cbe15470161.png)

这里对上面执行计划做下解释：首先条件 a>9000 使用了索引（关注 key 字段对应的值为 idx_a_b）；在 Extra 中，看到了“Using filesort”，
表示使用了 filesort 排序，并没有使用索引排序。所以联合索引中前面的字段使用了范围查询，对后面的字段排序使用不了索引排序。

原因是：a、b 两个字段的联合索引，对于单个 a 的值，b 是有序的。而对于 a 字段的范围查询，也就是 a 字段会有多个值，取到 a，b 的值 b 就
不一定有序了，因此要额外进行排序。联合索引结果如下图（为了便于理解，该图的值与上面所创建的表 t1 数据不一样）：

![](https://img.mukewang.com/5d3aaac80001701f10940513.png)

如上图所示，对于有 a、b 两个字段联合索引的表，如果对 a 字段范围查询，b 字段整体来看是无序的（如上图 b 的值为：1，2，3，1，2，
3······）。
##### ASC 和 DESC 混合使用将无法使用索引

对联合索引多个字段同时排序时，如果一个是顺序，一个是倒序，则使用不了索引，如下例：

```yaml
explain select id,a,b from t1 order by a asc,b desc;
```
![](https://img.mukewang.com/5d3aaa830001912015160158.png)

## group by 优化

默认情况，会对 group by 字段排序，因此优化方式与 order by 基本一致，如果目的只是分组而不用排序，可以指定 order by null 禁止排序。

## 总结
这次我们分享了 order by 和 group by 的一些优化技巧，下面我们一起来回顾下：

首先说到 MySQL 的两种排序方式：
- 通过有序索引直接返回有序数据
- 通过 Filesort 进行排序

建议优先考虑索引排序。

而Filesort又分为两种：
- 内存排序
- 磁盘文件排序

优先考虑内存排序。

Filesort 有三种排序模式：
- < sort_key, rowid >
- < sort_key, additional_fields >
- < sort_key, packed_additional_fields >

order by 语句的优化，这个是本节的重点：

- 通过添加合适索引
- 去掉不必要的返回字段
- 调整参数：主要是 max_length_for_sort_data 和 sort_buffer_size
- 避免几种无法利用索引排序的情况

>最后说到 group by 语句的优化，如果只要分组，没有排序需求的话，可以加 order by null 禁止排序。

