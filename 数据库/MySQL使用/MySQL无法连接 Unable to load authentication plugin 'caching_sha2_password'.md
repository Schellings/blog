---
title: MySQL无法连接 Unable to load authentication plugin 'caching_sha2_password'
date: 2018-02-16 16:56:24
tags: MySQL
---
# 问题分析
这个是因为，mysql8之前的版本使用的密码加密规则是mysql_native_password，但是在mysql8则是caching_sha2_password，所以需要修改密码加密规则。

# 问题解决
1、登录进入MySQL

2、执行以下命令：


```
use mysql;   
select user,host,plugin,authentication_string from user;
```
可以看到加密规则：

![image](https://img-blog.csdn.net/20180820163255821?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjE5MTgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

3、修改加密规则：

```
alter user 'root' @'localhost' identified with mysql_native_password by 'admin';
```
4、即可看到root的plugin 修改成功了

![image](https://img-blog.csdn.net/20180820163542192?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L20wXzM3NjE5MTgz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
