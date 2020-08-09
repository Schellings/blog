---
title: Redis实现用户热词推荐
date: 2019-01-03 19:56:24
tags: 
    - Redis
---
<meta name="referrer" content="no-referrer" />

## 数据结构设计

我们考虑对于系统来说 需要进行排序 因此考虑使用有序集合 即zset，我们需要先来设计对应存储的key，我们设计如下的key:
```yaml
f6car:brand:hot:123456789:qixiaobo
```

通常来说redis建议使用:来做为key的切分 我们以f6car:brand:hot作为前缀

123456789为对应的idOwnOrg ,qixiaobo为用户名

当用户对于carzone品牌进行点击时我们执行一个异步请求为:

```yaml
ZINCRBY f6car:brand:hot:111111:xielin 1 carzone
```

这个不需要考虑有否初始值 比如我们连续执行多次后结果如下:

```yaml
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "1"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "2"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "3"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "4"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "5"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "6"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 carzone
    "7"

    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 f6car
    "1"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 f6car
    "2"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 f6car
    "3"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 f6car
    "4"
    127.0.0.1:6379> ZINCRBY f6car:brand:hot:123456789:qixiaobo 1 f6car
    "5"
```

而对于mysql我们需要先查询再插入很容易造成并发问题 出现两个同样的品牌

当我们需要获取对应用户的点击情况只需要:

```yaml
    127.0.0.1:6379> ZREVRANGE f6car:brand:hot:123456789:qixiaobo 0 -1
    1) "carzone"
    2) "f6car"
```

其中-1 表示不限制长度【即全部数据】

当我们需要查询个数的限制时我们可以:

```yaml
    127.0.0.1:6379> ZREVRANGE f6car:brand:hot:123456789:qixiaobo 0 0
    1) "carzone"
```
如果需要返回分数我们也可以使用:

```yaml
    127.0.0.1:6379> ZREVRANGE f6car:brand:hot:123456789:qixiaobo 0 0 WITHSCORES
    1) "carzone"
    2) "7"
```


