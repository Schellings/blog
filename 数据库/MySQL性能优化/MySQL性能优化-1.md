---
title: MySQL性能优化（一）
date: 2019-02-05 16:56:24
tags: MySQL
---
# 1、 MySQL现状
MySQL 应该是最流行了 Web 后端数据库。Web 开发语言近期发展非常快，PHP， Ruby, Python, Java 各有特点，尽管 NOSQL 近期越來越多的被提到，可是相信大部分架构师还是会选择 MYSQL 来做数据存储。

MySQL 如此方便和稳定。以至于我们在开发 WEB 程序的时候非常少想到它。即使想到优化也是程序级别的，比方。不要写过于消耗资源的 SQL 语句。可是除此之外，在整个系统上仍然有非常多能够优化的地方。

# 2、优化策略
## 2、1 选择合适的存储引擎: InnoDB

除非你的数据表使用来做仅仅读或者全文检索 (相信如今提到全文检索，没人会用 MYSQL 了)。你应该默认选择 InnoDB 。

你自己在測试的时候可能会发现 MyISAM 比 InnoDB 速度快。这是由于： MyISAM 仅仅缓存索引，而 InnoDB 缓存数据和索引，MyISAM 不支持事务。可是 假设你使用 innodb_flush_log_at_trx_commit = 2 能够获得接近的读取性能 (相差百倍) 。

### 2.1.1、怎样将现有的 MyISAM 数据库转换为 InnoDB？

```
ALTER TABLE {tableName} ENGINE=InnoDB;
```
### 2.1.2 为每一个表分别创建 InnoDB FILE：

```
innodb_file_per_table=1
```
这样能够保证 ibdata1 文件不会过大。失去控制。尤其是在运行 mysqlcheck -o –all-databases 的时候。

## 2.2、 保证从内存中读取数据，将数据保存在内存中。

### 2.2.1、 设置足够大的 innodb_buffer_pool_size

#### 什么是innodb buffer pool
在MySQL5.5之前，广泛使用的和默认的存储引擎是MyISAM。MyISAM使用操作系统缓存来缓存数据。InnoDB需要innodb buffer pool中处理缓存。所以非常需要有足够的InnoDB buffer pool空间。

推荐将数据全然保存在 innodb_buffer_pool_size ，即按存储量规划 innodb_buffer_pool_size 的容量。

这样你能够全然从内存中读取数据。最大限度降低磁盘操作。

 #### InnoDB buffer pool 里包含什么
- 数据缓存: InnoDB数据页面

- 索引缓存: 索引数据

- 缓冲数据: 脏页（在内存中修改尚未刷新(写入)到磁盘的数据）

- 内部结构: 如自适应哈希索引，行锁等。

#### innodb_buffer_pool_size的取值

- innodb_buffer_pool_size默认大小为128M。最大值取决于CPU的架构。在32-bit平台上，最大值为2^32 -1,在64-bit平台上最大值为2^64-1。

- 当缓冲池大小大于1G时，将innodb_buffer_pool_instances设置大于1的值可以提高服务器的可扩展性。

- 大的缓冲池可以减小多次磁盘I/O访问相同的表数据。在专用数据库服务器上，可以将缓冲池大小设置为服务器物理内存的80%。

- InnoDB为缓冲区和control structures保留了额外的内存，因此总分配空间比指定的缓冲池大小大约大10％。

- 缓冲池大小必须始终等于或者是innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances的倍数。

- 如果将缓冲池大小更改为不等于或等于

**innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances**
 
 的倍数的值，则缓冲池大小将自动调整为等于或者是
 
 **innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances**
 
 的倍数的值。
 
- innodb_buffer_pool_size可以动态设置，允许在不重新启动服务器的情况下调整缓冲池的大小。

#### innodb_buffer_pool_size配置示例

在以下示例中

```
innodb_buffer_pool_size设置为3G;

innodb_buffer_pool_instances设置为8;

innodb_buffer_pool_chunk_size默认值为128M。
```


3G是有效的innodb_buffer_pool_size值，因为3G是innodb_buffer_pool_instances = 8 * innodb_buffer_pool_chunk_size = 128M的倍数。


```
# mysqld --innodb_buffer_pool_size=3G --innodb_buffer_pool_instances=8 &

mysql> show variables like 'innodb_buffer_pool%';

+-------------------------------------+----------------+
| Variable_name                       | Value          |
+-------------------------------------+----------------+
| innodb_buffer_pool_chunk_size       | 134217728      |
| innodb_buffer_pool_dump_at_shutdown | ON             |
| innodb_buffer_pool_dump_now         | OFF            |
| innodb_buffer_pool_dump_pct         | 25             |
| innodb_buffer_pool_filename         | ib_buffer_pool |
| innodb_buffer_pool_instances        | 8              |
| innodb_buffer_pool_load_abort       | OFF            |
| innodb_buffer_pool_load_at_startup  | ON             |
| innodb_buffer_pool_load_now         | OFF            |
| innodb_buffer_pool_size             | 3221225472     |
+-------------------------------------+----------------+
rows in set (0.01 sec)
```
在以下示例中

```
innodb_buffer_pool_size设置为3G；
innodb_buffer_pool_instances设置为16；
innodb_buffer_pool_chunk_size为128M。
```


3G不是有效的innodb_buffer_pool_size值，因为3G不是innodb_buffer_pool_instances = 16 * innodb_buffer_pool_chunk_size = 128M的倍数。

可以看出innodb_buffer_pool_size的值自动调整到4GB。


```
# mysqld --innodb_buffer_pool_size=3G --innodb_buffer_pool_instances=16 &

mysql> show variables like '%innodb_buffer_pool%';
+-------------------------------------+----------------+
| Variable_name                       | Value          |
+-------------------------------------+----------------+
| innodb_buffer_pool_chunk_size       | 134217728      |
| innodb_buffer_pool_dump_at_shutdown | ON             |
| innodb_buffer_pool_dump_now         | OFF            |
| innodb_buffer_pool_dump_pct         | 25             |
| innodb_buffer_pool_filename         | ib_buffer_pool |
| innodb_buffer_pool_instances        | 16             |
| innodb_buffer_pool_load_abort       | OFF            |
| innodb_buffer_pool_load_at_startup  | ON             |
| innodb_buffer_pool_load_now         | OFF            |
| innodb_buffer_pool_size             | 4294967296     |
+-------------------------------------+----------------+
rows in set (0.01 sec)
```

#### 在线调整InnoDB缓冲池大小

```
mysql> SET GLOBAL innodb_buffer_pool_size = 3221225472
```

#### 监控在线缓冲池调整进度


```
mysql> SHOW STATUS WHERE Variable_name='InnoDB_buffer_pool_resize_status';
+----------------------------------+----------------------------------------------------+
| Variable_name                    | Value                                              |
+----------------------------------+----------------------------------------------------+
| Innodb_buffer_pool_resize_status | Completed resizing buffer pool at 180824 15:05:03. |
+----------------------------------+----------------------------------------------------+
```



