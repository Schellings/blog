---
title: MySQL查询优化（一）
date: 2019-02-07 16:56:24
tags: MySQL
---

# 以下介绍几种常用的SQL查询优化方法

**对查询进行优化，应尽量避免全表扫描**
- 1、首先应考虑在 where 及 order by 涉及的列上建立索引。
- 2、应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描。


```
如：
select id from t where num is null
```
修改策略：

可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：

```
select id from t where num=0
```
- 3、应尽量避免在 where 子句中使用!=或<>操作符，否则将引擎放弃使用索引而进行全表扫描。

- 4、应尽量避免在 where 子句中使用 or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描。

```
如：
select id from t where num=10 or num=20
```
修改策略：

```
select id from t where num=10
union all
select id from t where num=20
```
- 5、in 和 not in 也要慎用，否则会导致全表扫描。


```
如：
select id from t where num in(1,2,3)
```

修改策略：


```
对于连续的数值，能用 between 就不要用 in 了：
select id from t where num between 1 and 3
```
- 6、下面的查询也将导致全表扫描：

```
select id from t where name like '%abc%'
```
若要提高效率，可以考虑全文检索引擎。

- 7、如果在 where 子句中使用参数，也会导致全表扫描。

因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。

然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```
select id from t where num=@num
```
修改策略：

```
可以改为强制查询使用索引：

select id from t with(index(索引名)) where num=@num
```
- 8、应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。


```
如：

select id from t where num/2=100
```
修改策略：


```
select id from t where num=100*2
```

- 9、应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。


```
如：

select id from t where substring(name,1,3)='abc'
--name以abc开头的id

select id from t where datediff(day,createdate,'2019-6-21')=0
--‘2019-6-21’生成的id
```
修改策略：

　
```
select id from t where name like 'abc%'

select id from t where createdate>='2005-11-30' and createdate<'2005-12-1'
```

- 10、不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。