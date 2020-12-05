---
title: Redis雪崩，穿透，击穿三连问
date: 2020-09-22 19:56:24
tags: 
    - Redis
    - 分布式
---
<meta name="referrer" content="no-referrer" />

## 引言
关于Redis雪崩，穿透，击穿的问题，第一次接触名字有点陌生，听上去还比较相似，难以理解，过去做的很多项目中也都是用过Redis，但是第一次听到这几个关于Redis的坑还是一脸懵逼，直到这些坑真正显灵的时候才彻底意识到要搞明白。

## Redis 雪崩

雪崩就是指**缓存中大批量热点数据过期后系统涌入大量查询请求**，因为大部分数据在Redis层已经失效，请求渗透到数据库层，大批量请求犹如洪水一般涌入，引起数据库压力造成查询堵塞甚至宕机。

**解决办法**：

- 将缓存失效时间分散开，比如每个key的过期时间是随机，防止同一时间大量数据过期现象发生，这样不会出现同一时间全部请求都落在数据库层，如果缓存数据库是分布式部署，将热点数据均匀分布在不同Redis和数据库中，有效分担压力，别一个人扛。

- 简单粗暴，让Redis数据永不过期（如果业务准许，比如不用更新的名单类）。当然，如果业务数据准许的情况下可以，比如中奖名单用户，每期用户开奖后，名单不可能会变了，无需更新。

## Redis 穿透

穿透是指绕过Reids，**调用者发起的请求参数（key）在缓存和数据库中都不存在**，通过不存在的key，成功穿透到系统底层，大规模不断发起不存在的key检索请求导致系统压力过大最后故障。

造成穿透的伪代码多为这样：

```java
if(redis.get(key) == null){
  // redis数据不存在或已经过期 查询数据库
 Object value = dao.query(key);
  // 重新将value刷入缓存。
  redis.set(key);
} else {
  return reids.get(key);
}
```

**解决方法**：

 - 分布式布隆过滤器：布隆是BloomFilter音译过来的，Redis 自身支持BloomFilter。
 - 返回空值：遇到数据库和Redis都查询不到的值，在Redis里set一个null value，过期时间很短，目的在于同一个key再次请求时直接返回null，避免穿透。
 
 伪代码：
 
 ```java
try {
  Object value = dao.query(key); // 从数据库中查询数据
  if (value == null) {
    redis.set(key, null, 20);   // set null值，10ms过期时间
  } else {
    redis.set(key, value, 1000);
  }
} catch(Exception e) {
  redis.set(key, null, 20);
}
```

## Redis 击穿

击穿和穿透概念类似，**一般是指一个key被穿透，这个key是热点key，同一个key会被有成千上万次请求**，比如微博热点排行榜，key是小时时间戳，value是个list的榜单。每个小时产生一个key，这个key会有百万QPS，如果这个key失效了，就像保险丝熔断，百万QPS直接压垮数据库。

**解决办法**:

业界比较常用的做法，是使用mutex。简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。类似下面的代码：

```java
public String get(key) {
    String value = redis.get(key);
    if (value == null) { //代表缓存值过期
        //设置3min的超时，防止del操作失败的时候，下次缓存过期一直不能load db
        if (redis.setnx(key_mutex, 1, 3 * 60) == 1) {  //代表设置成功
            value = db.get(key);
            redis.set(key, value, expire_secs);
            redis.del(key_mutex);
            } else {  //这个时候代表同时候的其他线程已经load db并回设到缓存了，这时候重试获取缓存值即可
            sleep(50);
            get(key);  //重试
            }
        } else {
            return value;      
        }
}
```

## 布隆过滤器原理

BloomFilter：即布隆过滤器。可以用于检索一个元素是否在一个集合中。

### 特点

BloomFilter检索一个元素是否在一个集合中有一定的错误率（很低），但不会漏判。

- 如果判断一个key不在集合中，那一定不在。
- 如果判断一个key存在，那不一定真的在。

本质上布隆过滤器是一种数据结构，比较巧妙的概率型数据结构（probabilistic data structure），特点是高效地插入和查询，可以用来告诉你 **“某样东西一定不存在或者可能存在”**。

相比于传统的 List、Set、Map 等数据结构，它更高效、占用空间更少，但是缺点是其返回的结果是概率性的，而不是确切的。

### 实现原理

基本思想是利用一个足够好的Hash函数将一个字符串映射到二进制位数组中的某一位，这样不管字符串如何长，都只有一位，因此存储空间就极大的提升了。但是不管hash函数如何高效，总是会存在Hash冲突，尤其是数据量变大的时候，而BloomFilter是利用多个不同的Hash函数来解决“冲突”，即一次采用多个Hash函数把数据映射到不同的位上，降低了冲突概率。如下图所示，



