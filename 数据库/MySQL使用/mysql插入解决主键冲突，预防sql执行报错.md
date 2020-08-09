---
title: MySQL插入解决主键冲突，预防sql执行报错
date: 2019-03-10 16:56:24
tags: MySQL
---
<meta name="referrer" content="no-referrer" />

三种方案：

- 使用 WHERE NOT EXISTS 检查是否存在
- 使用 ON DUPLICATE KEY UPDATE 在违反主键约束时更新 
- 使用 REPLACE 替换 INSERT  

```yaml
INSERT INTO `idm_app`
SELECT 'iam', 'IAM应用', 'default', 'default', 1, NULL, NULL, NULL, NULL, '2019-07-17 16:36:31', NULL, '2019-07-17 16:36:31', NULL, '{}',NULL
FROM DUAL
WHERE NOT EXISTS(SELECT * FROM `idm_app` WHERE id = 'iam')


INSERT INTO `idm_app` VALUES ('iam', 'IAM应用', 'default', 'default', 1, NULL, NULL, NULL, NULL, '2019-07-17 16:36:31', NULL, '2019-07-17 16:36:31', NULL, '{}',NULL) ON DUPLICATE KEY UPDATE `updated_at` = NOW();

REPLACE INTO `idm_app` VALUES ('iam', 'IAM应用', 'default', 'default', 1, NULL, NULL, NULL, NULL, '2019-07-17 16:36:31', NULL, '2019-07-17 16:36:31', NULL, '{}',NULL)
```


这三种方案，各有优缺点

第一种较为通用，不过你要是没有主键的话也只能用他了。

第二种和第三种的区别是：

1)insert是先尝试插入，若主键存在则更新。REPLACE是先尝试插入，若主键存在则删除原纪录再插入。

2)如果有多个唯一关键字发生冲突(不同关键字的冲突发生在不同记录),比如现在有2个字段2条记录冲突了(没条记录冲突一个字段)，
则insert是选择排序后在前面的一条进行更新，REPLACE是删除那两条记录，然后插入新记录。

> ON DUPLICATE KEY UPDATE vs REPLACE

1）没有key的时候，replace与insert .. on deplicate udpate相同。

2）有key的时候，都保留主键值，并且auto_increment自动+1

不同之处：

有key的时候，replace是delete老记录，而录入新的记录，所以原有的所有记录会被清除，这个时候，如果replace语句的字段不全的话，有些原有的比如c字段的值会被自动填充为默认值。
而insert .. deplicate update则只执行update标记之后的sql，从表象上来看相当于一个简单的update语句。
但是实际上，根据我推测，如果是简单的update语句，auto_increment不会+1，应该也是先delete，再insert的操作，只是在insert的过程中保留除update后面字段以外的所有字段的值。
所以两者的区别只有一个，insert .. on deplicate udpate保留了所有字段的旧值，再覆盖然后一起insert进去，而replace没有保留旧值，直接删除再insert新值。
从底层执行效率上来讲，replace要比insert .. on deplicate update效率要高，但是在写replace的时候，字段要写全，防止老的字段数据被删除。