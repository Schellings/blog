---
title: mysql重启后表丢失问题定位
date: 2020-10-09 19:56:24
tags: MySQL
---
## 问题描述

一次园区停电后，公司内部测试云环境断电关闭，上班恢复云平台后，发现目标数据库实例启动后表缺失，由于该表为核心业务表，需立即处理恢复正常业务。

## 问题现状

- mysql版本5.6.20，数据库重启后表丢失

- 重建表失败

ERROR 1022 (23000): Can't write; duplicate key in table 't_user'

- 查询表失败

```sqlite-psql
mysql> show create table t_user;
ERROR 1146 (42S02): Table 'test.t_user' doesn't exist
```


## 问题分析

- 表丢失前，数据库近期未进行数据库升级工作，无运维人员、开发测试操作过，排除人工操作导致的表丢失问题
- 表丢失前数据库实例断电重启过，怀疑是断电重启引发的一些故障问题
- 查看数据库启动日志

```xml
[Warning] InnoDB: Load table 'test/t_user' failed, the table has missing foreign key indexes. Turn off 'foreign_key_checks' and try again.
[Warning] InnoDB: Cannot open table test/t_user from the internal data dictionary of InnoDB though the .frm file for the table exists. See http://dev.mysql.com/doc/refman/5.6/en/innodb-troubleshooting.html for how you can resolve the problem.
```
- 根据启动日志分析，启动确实抛出警告信息，启动时数据库默认进行了数据的一致性检查，t_user表的外键检查失败，外键检查失败后，导致该表的数据无法加载到数据库实例中。
- 查看官方文档，确定问题影响范围和解决方案

[bug记录](https://bugs.mysql.com/bug.php?id=68148)
![img_1.png](img_1.png)

经查找，在mysql 5.6.12版本的发布更新中看到这个bug已经被fixed

[bug修复](https://dev.mysql.com/doc/relnotes/mysql/5.6/en/news-5-6-12.html)
![img.png](img.png)
>虽然官方在5.6.12给了bug修复记录，但是看bug反馈记录，在mysql 5.6.27版本该问题依然存在。

## 解决方案
经测试，当前5.6.20版本测试后确认问题依然存在。在mysql 5.7.26问题被修复。

- 修改mysql配置文件设置 `foreign_key_checks=0`，重启数据库实例 [参考](https://dev.mysql.com/doc/refman/5.6/en/server-system-variables.html#sysvar_foreign_key_checks)
- 重新连接数据库，发现缺失的表已经能正常连接
- 按照表外键规范，重建索引或删除脏数据














