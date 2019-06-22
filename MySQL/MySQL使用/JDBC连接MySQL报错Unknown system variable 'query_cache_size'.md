---
title: JDBC连接MySQL报错Unknown system variable 'query_cache_size'
date: 2019-02-18 16:56:24
tags: MySQL
---
# 问题描述
如题，在使用jdbc连接MySQL时报错，出现了一个

```
java.sql.SQLException: Unknown system variable 'query_cache_size'
```

# 问题解决

该问题的出现，是MySQL JDBC驱动jar包与MySQL版本不兼容，MySQL JDBC驱动jar包版本过旧。

我使用的是mysql 8版本，数据库驱动使用的是com.mysql.cj.jdbc.Driver 6.0
更换驱动版本至8.0.11，问题得到解决。


根据官方的说法是 :

The query cache is deprecated as of MySQL 5.7.20, and is removed in MySQL 8.0. Deprecation includes query_cache_size.

意思是query cache在MySQL5.7.20就已经过时了，而在MySQL8.0之后就已经被移除了。
