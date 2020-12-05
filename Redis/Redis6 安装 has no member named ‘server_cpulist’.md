---
title: Redis6 安装失败 【has no member named ‘server_cpulist’】
date: 2020-10-02 19:56:24
tags: 
    - Redis
---

<meta name="referrer" content="no-referrer" />

## 背景

Redis6.0新版本在五月初重磅发布，我们可以清晰地发现Redis6.0新版本引入了多线程。相信大家一定都十分好奇，Redis6.0引入多线程究竟有哪些好处呢？下面我们就来先安装尝鲜

## 安装

一如常规操作，此处redis使用make安装

```yaml
$ wget https://download.redis.io/releases/redis-6.0.9.tar.gz
$ tar xzf redis-6.0.9.tar.gz
$ cd redis-6.0.9
$ make
```

但是抛出了大量编译错误，如下

```yaml
server.c:5168:15: error: ‘struct redisServer’ has no member named ‘maxmemory’
     if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
               ^
server.c:5168:39: error: ‘struct redisServer’ has no member named ‘maxmemory’
     if (server.maxmemory > 0 && server.maxmemory < 1024*1024) {
                                       ^
server.c:5169:176: error: ‘struct redisServer’ has no member named ‘maxmemory’
         serverLog(LL_WARNING,"WARNING: You specified a maxmemory value that is less than 1MB (current value is %llu bytes). Are you sure this is what you really want?", server.maxmemory);
```

问题在于gcc版本过低，需升级至5.3版本以上

## gcc升级

```yaml
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
scl enable devtoolset-9 bash
-- 设置永久升级：
echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
```

## 构建

```yaml
make && make install
```

此时在{redis}/src下将构建出对应的可执行文件