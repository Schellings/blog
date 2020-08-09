---
title: Long源码解析
date: 2018-08-20 19:56:24
tags: 
    - 源码
    - Java
---
<meta name="referrer" content="no-referrer" />
## Long
### 缓存
Long 最被我们关注的就是 Long 的缓存问题，Long 自己实现了一种缓存机制，缓存了从
-128 到 127 内的所有 Long 值，如果是这个范围内的 Long 值，就不会初始化，而是从缓存
中拿，缓存初始化源码如下：

```java
private static class LongCache {
    private LongCache(){}
    // 缓存，范围从 -128 到 127，+1 是因为有个 0
    static final Long cache[] = new Long[-(-128) + 127 + 1];

    // 容器初始化时，进行加载
    static {
        // 缓存 Long 值，注意这里是 i - 128 ，所以再拿的时候就需要 + 128
        for(int i = 0; i < cache.length; i++)
            cache[i] = new Long(i - 128);
    }
}
```

为什么使用 Long 时，大家推荐多使用 valueOf 方法，少使用 parseLong 方法？

答：因为 Long 本身有缓存机制，缓存了 -128 到 127 范围内的 Long，valueOf 方法会从缓
存中去拿值，如果命中缓存，会减少资源的开销，parseLong 方法就没有这个机制。