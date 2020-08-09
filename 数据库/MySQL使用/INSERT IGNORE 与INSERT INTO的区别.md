---
title: INSERT IGNORE 与INSERT INTO的区别
date: 2020-02-08 16:56:24
tags: MySQL
---
<meta name="referrer" content="no-referrer" />
## 三种Mysql Insert语法
INSERT IGNORE 与INSERT INTO的区别就是 INSERT IGNORE会忽略数据库中已经存在 的数据，如果数据库没有数据，就插入新的数据，如果有数据的话就跳过这条数据。这样就可以保留数据库中已经存在数据，达到在间隙中插入数据的目的。

eg: insert ignore into table(name)  select  name from table2 

mysql中常用的三种插入数据的语句:

- insert into表示插入数据，数据库会检查主键（PrimaryKey），如果出现重复会报错；

- replace into表示插入替换数据，需求表中有PrimaryKey，或者unique索引的话，如果数据库已经存在数据，则用新数据替换，如果没有数据效果则和insert into一样；

```yml
REPLACE语句会返回一个数，来指示受影响的行的数目。该数是被删除和被插入的行数的和。
如果对于一个单行REPLACE该数为1，则一行被插入，同时没有行被删除。如果该数大于1，则在新行被插入前，有一个或多个旧行被删除。
如果表包含多个唯一索引，并且新行复制了在不同的唯一索引中的不同旧行的值，则有可能是一个单一行替换了多个旧行。
```

- insert ignore表示，如果中已经存在相同的记录，则忽略当前新数据；

## 案例

```sql
create table testtb(
  id int not null primary key,
  name varchar(50),
  age int
);

insert into testtb(id,name,age)values(1,"bb",13);
select * from testtb; # 查询出一条记录

insert ignore into testtb(id,name,age)values(1,"aa",13); # 触发主键冲突，忽略本次插入
select * from testtb;//查询出一条记录，仍是1，bb,13

replace into testtb(id,name,age) values (1,"aa",12); # 触发注解约束，产生替换更新动作
select * from testtb; ////查询出一条记录，数据变为1,aa,12
```