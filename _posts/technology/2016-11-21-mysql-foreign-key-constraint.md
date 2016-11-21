---
layout: post
title: MySQL 三种外键约束方式
category: 技术
tags: MySQL
keywords: MySQL 
description: MySQL 三种外键约束方式
---

## 准备工作

我们以学生表和班级表为例，一个班级对于多个学生。Windows 系统 cnd 命令行环境下进行。

- 数据库创建：

```
create database ins;
```
- 班级表创建：

```
create table ins_class(
    id int not null ,
    name varchar(32),
    primary key(id)
);
```

- 插入数据：

```
insert into ins_class values (1,'classA'),(2,'classB');
```
下面创建学生表，分别以不同的约束方式创建外键引用关系：

## 级联(cascade)方式

```
create table ins_students (
    id int not null,
    name varchar(32),
    class_id int,
    primary key (id),
    foreign key (class_id) references ins_class(id) on delete cascade on update cascade
);
```

一、 参照完整性测试

```
insert into ins_students values(1,'xiaohai',1);
    #Query OK, 1 row affected (0.04 sec)

insert into ins_students values(2,'xiaohua',2);
    #Query OK, 1 row affected (0.05 sec)

insert into ins_students values(3,'xiaoxiao',3);
    #ERROR 1452 (23000): Cannot add or update a child row: 
    a foreign key constraint fails (`ins`.`ins_students`,
    CONSTRAINT `ins_students_ibfk_1` FOREIGN KEY (`class_id`) 
    REFERENCES `ins_class` (`id`) ON DELETE CASCADE ON UPDATE CASCADE)
```

二、 约束方式测试 

```
insert into ins_students values(1,'xiaohai',1);
insert into ins_students values(2,'xiaohua',2);
insert into ins_students values(3,'xiaoxiao',2);

delete from ins_class where id=2;
Query OK, 1 row affected (0.08 sec)

select * from ins_students;
+----+---------+----------+
| id | name    | class_id |
+----+---------+----------+
|  1 | xiaohai |        1 |
+----+---------+----------+

update ins_class set id=10 where id=1;
    #Query OK, 1 row affected (0.04 sec)
    #Rows matched: 1  Changed: 1  Warnings: 0

select * from ins_students;
+----+---------+----------+
| id | name    | class_id |
+----+---------+----------+
|  1 | xiaohai |       10 |
+----+---------+----------+

```

### 置空(set null)方式

```
create table ins_students (
    id int not null,
    name varchar(32),
    class_id int,
    primary key (id),
    foreign key (class_id) references ins_class(id) on delete set null on update set null
);
```

一、 参照完整性测试

```
insert into ins_students values (1,'xiaohai',1);
Query OK, 1 row affected (0.04 sec)

insert into ins_students values (2,'xiaohua',2);
Query OK, 1 row affected (0.07 sec)

insert into ins_students values (3,'xiaohua',3);
ERROR 1452 (23000): Cannot add or update a child row: 
a foreign key constraint fails (`ins`.`ins_students`, CONSTRAINT 
`ins_students_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES 
`ins_class` (`id`) ON DELETE SET NULL ON UPDATE SET NULL)
```

二、 约束方式测试

```
insert into ins_students values(1,'xiaohai',1);
insert into ins_students values(2,'xiaohua',2);

delete from ins_class where id=2;
Query OK, 1 row affected (0.04 sec)

select * from ins_students;
+----+---------+----------+
| id | name    | class_id |
+----+---------+----------+
|  1 | xiaohai |        1 |
|  2 | xiaohua |     NULL |
+----+---------+----------+
2 rows in set (0.00 sec)

update ins_class set id=10 where id=1;
Query OK, 1 row affected (0.04 sec)
Rows matched: 1  Changed: 1  Warnings: 0

select * from ins_class;
+----+--------+
| id | name   |
+----+--------+
| 10 | classA |
+----+--------+
1 row in set (0.00 sec)

select * from ins_students;
+----+---------+----------+
| id | name    | class_id |
+----+---------+----------+
|  1 | xiaohai |     NULL |
|  2 | xiaohua |     NULL |
+----+---------+----------+
2 rows in set (0.00 sec)

```

### 禁止(no action/ restrict)方式

```
create table ins_students (
    id int not null,
    name varchar(32),
    class_id int,
    primary key (id),
    foreign key (class_id) references ins_class(id) on delete no action on update no action
);
```

一、参照完整性测试

```
insert into ins_students values (1,'xiaohai',1);
Query OK, 1 row affected (0.05 sec)

insert into ins_students values (2,'xiaohua',2);
Query OK, 1 row affected (0.05 sec)

insert into ins_students values (3,'xiaoxiao',3);
ERROR 1452 (23000): Cannot add or update a child row: 
a foreign key constraint fails (`ins`.`ins_students`,
CONSTRAINT `ins_students_ibfk_1` FOREIGN KEY (`class_id`)
REFERENCES `ins_class` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION)
```
二、约束方式测试

```
insert into ins_students values(1,'xiaohai',1);
insert into ins_students values(2,'xiaohua',2);
insert into ins_students values(3,'xiaoxiao',2);

delete from ins_class where id=2;
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`ins`.`ins_students`, CONSTRAINT `ins_students_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES `ins_class` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION)

update ins_class set id=10 where id=1;
ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`ins`.`ins_students`, CONSTRAINT `ins_students_ibfk_1` FOREIGN KEY (`class_id`) REFERENCES `ins_class` (`id`) ON DELETE NO ACTION ON UPDATE NO ACTION)

```

在 MySQL 中，restrict 方式与 no action 方式作用相同。









