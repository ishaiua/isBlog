---
layout: post
title: MySQL 优化
category: 技术
tags: MySQL
description: MySQL
---


## MySQL 优化方向

基于 MySQL 5.7 。

1. 存储层：数据表存储引擎选取（MyISAM、InnoDB等）、字段的选取、逆范式（3范式）。
2. 设计层：索引、分区/分表。
3. 架构层：分布式部署（主从模式）。
4. 语句层：编写高效率的语句。


## 存储层

### 数据库引擎

```php
mysql> show engines;
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| Engine             | Support | Comment                                                        | Transactions | XA   | Savepoints |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
| InnoDB             | DEFAULT | Supports transactions, row-level locking, and foreign keys     | YES          | YES  | YES        |
| MRG_MYISAM         | YES     | Collection of identical MyISAM tables                          | NO           | NO   | NO         |
| MEMORY             | YES     | Hash based, stored in memory, useful for temporary tables      | NO           | NO   | NO         |
| BLACKHOLE          | YES     | /dev/null storage engine (anything you write to it disappears) | NO           | NO   | NO         |
| MyISAM             | YES     | MyISAM storage engine                                          | NO           | NO   | NO         |
| CSV                | YES     | CSV storage engine                                             | NO           | NO   | NO         |
| ARCHIVE            | YES     | Archive storage engine                                         | NO           | NO   | NO         |
| PERFORMANCE_SCHEMA | YES     | Performance Schema                                             | NO           | NO   | NO         |
| FEDERATED          | NO      | Federated MySQL storage engine                                 | NULL         | NULL | NULL       |
+--------------------+---------+----------------------------------------------------------------+--------------+------+------------+
9 rows in set (0.00 sec)
```

常见数据库引擎特性：

- InnoDB：支持事务、行级锁、外键。
- MyISAM：速度快，可压缩。

### 字段选取

选择占据空间小的字段

- tinyint 1 字节 
- smallint 2 字节 
- mediumint 3 字节 
- int 4 字节 
- bigint 8 字节

长度固定的字段

- char 255 字符
- varchar 65535 字节

char 速度快，varchar 空间节省。

整形存储

例如 ip 地址通过整形存储。
mysql：inet_aton(ip) inet_ntoa (数字)
php: ip2long (ip) long2ip (数字)


### 逆范式

有时候为了避开联表查询过程中影响数据库查询速度，可以采取逆范式数据库设计。




## 设计层

### 索引

索引：是进行数据库设计的时候，提升性能最有效的一种技术（例如主键索引），有主键索引、唯一索引、普通索引、全文索引。

```
mysql> select count(*) from emp;
+----------+
| count(*) |
+----------+
|  1800000 |
+----------+
1 row in set (0.04 sec)

mysql> select * from emp where empno=1324765;
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
| empno   | ename  | job      | mgr | hiredate   | sal     | comm   | deptno | epassword                        |
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
| 1324765 | XgODwz | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |     84 | 0609de8e71ddfa6d2171ecd7e547c2f6 |
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
1 row in set (1.50 sec)

mysql> alter table emp add primary key(empno);
Query OK, 1800000 rows affected (6.40 sec)
Records: 1800000  Duplicates: 0  Warnings: 0

mysql> select * from emp where empno=1324765;
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
| empno   | ename  | job      | mgr | hiredate   | sal     | comm   | deptno | epassword                        |
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
| 1324765 | XgODwz | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |     84 | 0609de8e71ddfa6d2171ecd7e547c2f6 |
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
1 row in set (0.00 sec)


```

速度提升显著。

```
alter table student add  primary key (id);
alter table student add  unique key  (name);
alter table student add  key  (height);
alter table student add  fulltext  key (introduce);
alter table student add  key name_height (name,height); # 复合索引

alter table student modify  id int not null comment '主键';
alter table student modify id int  not null auto_increment comment '主键';
alter table student drop primary key; #删除索引   


show create table student;
```

为使用索引查询计划测试：

```
mysql> explain select * from emp where deptno=381\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: emp
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1800000
     filtered: 10.00
        Extra: Using where
1 row in set, 1 warning (0.05 sec)
```

使用主键查询：

```
mysql> explain select * from emp where empno=1342567\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: emp
   partitions: NULL
         type: const
possible_keys: PRIMARY
          key: PRIMARY
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
1 row in set, 1 warning (0.04 sec)
```

适合使用索引地方：

一、where 查询条件后边的字段。

```
mysql> explain select * from emp order by ename limit 1000000,5\G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: emp
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1800000
     filtered: 100.00
        Extra: Using filesort
1 row in set, 1 warning (0.00 sec)
```

排序字段适合做索引。

使用原则：

1. 字段独立原则，即没有计算。
2. 左原则，模糊查询，name% 可以，%name% 则不可以。
3. 复合索引，第二字段不可以单独作为查询条件进行索引。
4. or 原则，or 左右查询条件字段都有索引，则可以；只有一个有索引，则不可以。

### 设计依据

1. 平凡使用的字段。
2. 执行时间长的查询语句设置索引。
3. 逻辑非常重要的语句考虑索引。
4. 字段内容重复率高的不适合做索引。

### 前缀索引

某字段内容，通过前面几位数据就看可以确定整体内容的，这样的可以取前几位创建索引：

alter table 表名 add key （字段（位数））；


### 缓存设置

```
set globale query_cache_size=10*1024*1024;
```

## 分表设置

分表方式：
1. 逻辑分表： MySQL 自身具有的分表技术，该方式 PHP 逻辑不受影响。
2. 物理分表：需要增加算法逻辑。

### 逻辑分表设计

```
CREATE TABLE `goods_key` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) NOT NULL COMMENT '名称',
  `price` decimal(10,2) DEFAULT '0.00' COMMENT '价格',
  `published_at` date NOT NULL DEFAULT '0000-00-00' COMMENT '上架时间',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8
PARTITION by key(id) PARTITIONS 10;  

CREATE TABLE `goods_hash` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) NOT NULL COMMENT '名称',
  `price` decimal(10,2) DEFAULT '0.00' COMMENT '价格',
  `published_at` date NOT NULL DEFAULT '0000-00-00' COMMENT '上架时间',
  PRIMARY KEY (`id`,`published_at`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8
PARTITION by hash(month(published_at)) PARTITIONS 12;

CREATE TABLE `goods_range` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) NOT NULL COMMENT '名称',
  `price` decimal(10,2) DEFAULT '0.00' COMMENT '价格',
  `published_at` date NOT NULL DEFAULT '0000-00-00' COMMENT '上架时间',
  PRIMARY KEY (`id`,`published_at`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8
partition by range(year(published_at))(
partition 70s values less than (1980),
partition 80s values less than (1990),
partition 90s values less than (2000),
partition 00s values less than (2010),
partition `小屁孩` values less than (2020),
);

CREATE TABLE `goods_list` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) NOT NULL COMMENT '名称',
  `price` decimal(10,2) DEFAULT '0.00' COMMENT '价格',
  `published_at` date NOT NULL DEFAULT '0000-00-00' COMMENT '上架时间',
  PRIMARY KEY (`id`,`published_at`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8
partition by list(month(published_at))(
partition spring values in(2,3,4), 
partition summer values in(5,6,7),
partition autumn values in(8,9,10),
partition winter values in(11,12,1)
);


```

对已有数据表进行增加分表操作，数据不会丢失，减少分表，在 range 和 list 类会造成数据丢失。

```
alter table 表名 add partition partitions 数量;
alter table 表名 add  partitions(
partitions 名称 values less than 数量,
  or 
partitions 名称 values in (数量1，数量2，数量3)
);


alter table 表名 coalesce partition 12; ## key/hash 求余方式

alter table 表名 drop partition 分区名称;  ## range/list 范围方式

rename table student to students;  ##更改表名
```
### 垂直分表

水平分表：是把一个表的全部记录信息分表存储在不同的分表中；
垂直分表：是把一个表的全部字段分别存储在不同的分表中。

## 架构设计

集群设计，多台 MySQL 服务器共同支持服务，每台服务器分担的压力就比较小，运行速度就比较高（主从模式）。

### 慢查询日记收集

```
mysql> show variables like "slow_query%";
+---------------------+---------------------------------------------+
| Variable_name       | Value                                       |
+---------------------+---------------------------------------------+
| slow_query_log      | OFF                                         |
| slow_query_log_file | E:\wamp\MySQL\data\DESKTOP-OS72MA8-slow.log |
+---------------------+---------------------------------------------+
2 rows in set, 1 warning (0.07 sec)


mysql> set global slow_query_log=1;
Query OK, 0 rows affected (0.07 sec)

mysql> show variables like "slow_query%";
+---------------------+---------------------------------------------+
| Variable_name       | Value                                       |
+---------------------+---------------------------------------------+
| slow_query_log      | ON                                          |
| slow_query_log_file | E:\wamp\MySQL\data\DESKTOP-OS72MA8-slow.log |
+---------------------+---------------------------------------------+
2 rows in set, 1 warning (0.00 sec)

mysql> show variables like"long_query%";
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set, 1 warning (0.00 sec)

mysql> set long_query_time=2;
Query OK, 0 rows affected (0.00 sec)
```
执行查询：

```
mysql> select * from emp order by empno limit 1000000,5;
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
| empno   | ename  | job      | mgr | hiredate   | sal     | comm   | deptno | epassword                        |
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
| 1100002 | bTrdqv | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |     64 | f6acc1aecb525fb404ace3ca2fff3959 |
| 1100003 | pgOzhl | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |    331 | 6ddf0ce269928ec2e639c50e6b741fd0 |
| 1100004 | EYfJQl | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |    328 | db0766bca10e673bbbdf5cb612bc315f |
| 1100005 | BLbYNT | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |     71 | 4f077d37cc9153290586ead641a78aeb |
| 1100006 | ZCQuCE | SALESMAN |   1 | 2015-04-12 | 2000.00 | 400.00 |    154 | c9f6a8f77d3a29db9a57c4a72055da91 |
+---------+--------+----------+-----+------------+---------+--------+--------+----------------------------------+
5 rows in set (5.60 sec)
```
查看日记

```
MySQL, Version: 5.7.14 (MySQL Community Server (GPL)). started with:
TCP Port: 3306, Named Pipe: MySQL
Time                 Id Command    Argument
# Time: 2017-01-30T03:49:30.934858Z
# User@Host: root[root] @ localhost [::1]  Id:     3
# Query_time: 5.595896  Lock_time: 0.000000 Rows_sent: 5  Rows_examined: 2800005
use newtest;
SET timestamp=1485748170;
select * from emp order by empno limit 1000000,5;
```

### 批量插入数据


MyISAM

关闭/开启索引：
```
alter table 表名 disable keys;
alter table 表名 enable keys;

insert into 表名 values(),()...(); #m个
      .
      .
      .
      n-1

insert into 表名 values(),()...();  
 

```

InnoDB

无需对索引进行关闭处理，因为支持事务。可以先写数据，后维护索引。
开启事务：

start transaction；
insert 语句；
commit；

### 联表查询


可以单表进行查询，在通过 PHP 逻辑进行合表，有效提高并发性。

### 杂项

```
 select s.id,s.name,group_concat(c.cname separator ' ') from stu s join stu_course sc on s.id=sc.sid join course c on sc.cid=c.cid group by id;



+----+------+-------------------------------------+
| id | name | group_concat(c.cname separator ' ') |
+----+------+-------------------------------------+
|  1 | 小明 | 英语 数学 语文                       |
|  2 | 小红 | 数学 语文                           |
|  3 | 小张 | 语文                                |
+----+------+-------------------------------------+
```


## 总结

除以上常见的之外还有一些强制不排序之类的优化方式。










