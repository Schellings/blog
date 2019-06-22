---
title: 执行MySQL语句报错：the right syntax to use near xxx
date: 2019-02-16 16:56:24
tags: MySQL
---
# 问题产生的原因：
- SQL语句拼写错误
- 包含中文逗号，括号或问号等
- 对于MySQL 8.0所有的表名和表字段都需要在两边加上``

```
MySQL 5.7
insert into user(name,gender,phone) values('mark','男','1377777777');
或
insert into `user`(`name`,`gender`,`phone`) values('mark','男','1377777777');
MySQL 8.0
强制要求使用：
insert into `user`(`name`,`gender`,`phone`) values('mark','男','1377777777');
```
```
MySQL 5.7
select username,password from user where id='1';
或
select `username`,`password` from `user` where id='1';
或
select `username`,`password` from `user` where (id='1');

MySQL 8.0
select `username`,`password` from `user` where (id='1');
```
