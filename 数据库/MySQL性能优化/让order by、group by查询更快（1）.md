---
title: 让order by、group by查询更快（1）
date: 2018-12-17 16:56:24
tags: 
  - SQL优化
  - MySQL
---
<meta name="referrer" content="no-referrer" />
## 前言
在工作中，我们应该经常会遇到需要对查询的结果进行排序或者分组的情况。你是否会在意这两类 SQL 的执行效率呢？这篇文稿就一起讨论下如
何优化 order by 和 group by 语句。

## order by 原理

在优化 order by 语句之前，需要先了解 MySQL 中排序的相关知识点和原理，为了方便讲解过程举例说明，首先创建一张测试表，建表及数据写
入语句如下：

```yaml
CREATE TABLE `t1` (	      /* 创建表t1 */
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `a` int(20) DEFAULT NULL,
  `b` int(20) DEFAULT NULL,
  `c` int(20) DEFAULT NULL,
  `d` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_a_b` (`a`,`b`),
  KEY `idx_c` (`c`)
) ENGINE=InnoDB CHARSET=utf8mb4 ;

drop procedure if exists insert_t1; /* 如果存在存储过程insert_t1，则删除 */
delimiter ;;
create procedure insert_t1()        /* 创建存储过程insert_t1 */
begin
  declare i int;                  /* 声明变量i */
  set i=1;                        /* 设置i的初始值为1 */
  while(i<=10000)do			      /* 对满足i<=10000的值进行while循环 */
    insert into t1(a,b,c) values(i,i,i); /* 写入表t1中a、b两个字段，值都为i当前的值 */
    set i=i+1;                       /* 将i加1 */
  end while;
end;;
delimiter ;
call insert_t1();               /* 运行存储过程insert_t1 */

update t1 set a=1000 where id >9000;
```

下面一起来研究下 MySQL 的排序原理：

### MySQL 的排序方式
按照排序原理分，MySQL 排序方式分两种：

- 通过有序索引直接返回有序数据
- 通过 Filesort 进行的排序

>怎么确定某条排序的 SQL 所使用的排序方式？

使用 explain 来查看该排序 SQL 的执行计划，重点关注 Extra 字段： 如果该字段里显示是 Using index，则表示是通过有序索引直接返回有序数据。比如：

```yaml
explain select id,c from t1 order by c;
```

![](https://img.mukewang.com/5d3aacd700015e6b12830163.png)

如果该字段里显示是 Using filesort，则表示该 SQL 是通过 Filesort 进行的排序，比如：

```yaml
explain select id,d from t1 order by d;
```

![](https://img.mukewang.com/5d3aaca40001a87312870155.png)

那么 Filesort 是否都是在磁盘中完成排序操作的呢？下面我们再看看 Filesort 排序的详情。

### Filesort 排序的位置

即Filesort 是在内存中还是在磁盘中完成排序的？

MySQL 中的 Filesort 并不一定是在磁盘文件中进行排序的，也有可能在内存中排序，内存排序还是磁盘排序取决于排序的数据大小和 sort_buffer_size 配置的大小。

>如果 “排序的数据大小” < sort_buffer_size: 内存排序
 如果 “排序的数据大小” > sort_buffer_size: 磁盘排序
 
 怎么确定使用 Filesort 排序的 SQL 是在内存还是在磁盘中进行的排序操作？
 
 此时就可以使用 trace 进行分析，重点关注 number_of_tmp_files，
 如果等于 0，则表示排序过程没使用临时文件，在内存中就能完成排序；
 如果大于0，则表示排序过程中使用了临时文件。
 
 如下图，因为 number_of_tmp_files 等于 0，表示未使用临时文件进行排序，所以是内存排序。
 ![](https://img.mukewang.com/5d3aac770001419808410145.png)
 
 这里解释一下上面一些参数的含义：
 >rows：预计扫描的行数
  examined_rows：参与排序的行
  number_of_tmp_files：使用临时文件的个数
  sort_buffer_size：sort_buffer 的大小
  sort_mode：排序模式
  
  再看一个用到临时文件的例子，如下图，因为 number_of_tmp_files 等于 7，所以表示使用的是磁盘排序。
  
  对于 number_of_tmp_files 等于 7 表示该 SQL 将需要排序的数据分为 7 份，然后每份单独排序，再存放在 7 个临时文件中，最后把 7 个临时文件合并成一个大的有序文件。
  
  ![](https://img.mukewang.com/5d3aac470001298c07830146.png)
  
### Filesort 下的排序模式
  
  Filesort 下的排序模式有三种，具体介绍如下：（参考[《MySQL 5.7 Reference Manual》8.2.1.14 ORDER BY Optimization](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)）
  
  >三种排序方式：
  < sort_key, rowid >双路排序（又叫回表排序模式）：是首先根据相应的条件取出相应的排序字段和可以直接定位行数据的行 ID，然后在 sort buffer 中进行排序，排序完后需要再次取回其它需要的字段；
  < sort_key, additional_fields >单路排序：是一次性取出满足条件行的所有字段，然后在sort buffer中进行排序；
  < sort_key, packed_additional_fields >打包数据排序模式：与单路排序相似，区别是将 char 和 varchar 字段存到 sort buffer 中时，更加紧缩。
  

因为打包数据排序模式是单路排序的一种升级模式，因此重点探讨双路排序和单路排序的区别。

#### 选择策略

MySQL通过比较系统变量max_length_for_sort_data 的大小和需要查询的字段总大小来判断使用哪种排序模式。

- 如果 max_length_for_sort_data 比查询字段的总长度大，那么使用 < sort_key, additional_fields >排序模式；
- 如果 max_length_for_sort_data 比查询字段的总长度小，那么使用 <sort_key, rowid> 排序模式。

下面一起来通过实验验证参数 max_length_for_sort_data 对排序模式的影响：


```yaml
set session optimizer_trace="enabled=on",end_markers_in_json=on;
SET max_length_for_sort_data = 20;
select a,d from t1 order by d; /* 查询表t1的id、a、d三个字段的值，按照字段d进行排序 */
SELECT * FROM information_schema.OPTIMIZER_TRACE\G
```
![](https://img.mukewang.com/5d3aac0100016e5c07900149.png)

发现使用的排序模式是 < sort_key, additional_fields >

怎么让这条 SQL 的排序模式变成 <sort_key, rowid> 呢？下面我们来试验下：
因为 a、d 两个字段的总长度为 12，可以尝试把 max_length_for_sort_data 改为小于 12 的值，看排序模式是否有改变。

>知识扩展：MySQL 常见字段类型及所占字节。

字段类型 |字节
---|---
INT |4
BIGINT| 8
DECIMAL(M,D) |M+2
DATETIME |8
TIMESTAMP |4
CHAR(M) |M
VARCHAR(M)| M

```yaml
set session optimizer_trace="enabled=on",end_markers_in_json=on;
set max_length_for_sort_data = 4;
select a,d from t1 order by d;
SELECT * FROM information_schema.OPTIMIZER_TRACE\G
```

![](https://img.mukewang.com/5d3aabe900015f9a07240144.png)

可能讲到这里，你会有个疑问，为什么要添加 max_length_for_sort_data 这个参数让排序使用不同的排序模式呢？

限定只用一种排序模式不行吗？

接下来，我们一起分析下 max_length_for_sort_data 的重要性。比如下面这条 SQL：
```yaml
select a,c,d from t1 where a=1000 order by d;
```

我们先看**单路排序**的详细过程：

1. 从索引 a 找到第一个满足 a = 1000 条件的主键 id
2. 根据主键 id 取出整行，取出 a、c、d 三个字段的值，存入 sort_buffer 中
3. 从索引 a 找到下一个满足 a = 1000 条件的主键 id
4. 重复步骤 2、3 直到不满足 a = 1000
5. 对 sort_buffer 中的数据按照字段 d 进行排序
6. 返回结果给客户端

我们再看下**双路排序**的详细过程：

1. 从索引 a 找到第一个满足 a = 1000 的主键 id
2. 根据主键 id 取出整行，把排序字段 d 和主键 id 这两个字段放到 sort buffer 中
3. 从索引 a 取下一个满足 a = 1000 记录的主键 id
4. 重复 3、4 直到不满足 a = 1000
5. 对 sort_buffer 中的字段 d 和主键 id 按照字段 d 进行排序
6. 遍历排序好的 id 和字段 d，按照 id 的值回到原表中取出 a、c、d 三个字段的值返回给客户端

其实对比两个排序模式，单路排序会把所有需要查询的字段都放到 sort buffer 中，而双路排序只会把主键和需要排序的字段放到 sort buffer 中进
行排序，然后再通过主键回到原表查询需要的字段。

如果 MySQL 排序内存配置的比较小并且没有条件继续增加了，可以适当把 max_length_for_sort_data 配置小点，让优化器选择使用 rowid 排序
算法，可以在 sort_buffer 中一次排序更多的行，只是需要再根据主键回到原表取数据。

如果 MySQL 排序内存有条件可以配置比较大，可以适当增大 max_length_for_sort_data 的值，让优化器优先选择全字段排序，把需要的字段放
到 sort_buffer 中，这样排序后就会直接从内存里返回查询结果了。

所以 MySQL 通过 max_length_for_sort_data 这个参数来控制排序，在不同场景使用不同的排序模式，从而提升排序效率。













  
  