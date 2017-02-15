---
layout: post
title: MySQL 商品无限规格设计
category: 技术
tags: sku
description: 商品规格 sku
---

### 商品多规格

最近面试的时候遇到一个问题，大概意思是说商城一件衣服，颜色有蓝、白、黑，尺码有  S、M、L 码，不同的颜色和尺码搭配的价格和库存是不一样的，这种关系如何设计数据库呢？

  ![示例图](http://og6e4y8ws.bkt.clouddn.com/20170215-mysql.png)

之前也没用遇到过这种问题，第一感觉就是以衣服表为主表，再分表建立子表：颜色表和尺码表以及颜色-尺码管理表。

商品表字段(item)：id name...
颜色表字段(color)：id name item_id ...
尺码表字段（chima）:id name item_id ...
颜色-尺码表（c-c）:id item_id color_id chima_id number price ...

回来后上网搜索了相关的内容，发现其实早就有这种 库存 SKU 的文章和设计，关于上面的问题有如下的具体解决方案：

### 商品的无限规格实现

我们发现上面的衣服的库存量单位(SKU)便不再是该商品, 而是到具体属性组合出的规格, 每种规格可能会有不同的售价、运费与库存剩余情况, 所以用户在购买时, 不仅需要记录所购买的商品 ID, 同时也需要记录购买的该商品的具体规格 。

直观分析图示中的规格情况, 尺码、颜色属于衣服不同属性的名称, 与之对应的为属性可选择的的具体值, 属于一对多关系, 在 MySQL 数据库表结构中反应出为:

```
# 商品属性名
CREATE TABLE `item_attr_key` (
    `attr_key_id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键',     
    `item_id` INT(10) UNSIGNED NOT NULL COMMENT '商品ID',                      # 关联到商品
    `attr_name` VARCHAR(50) NOT NULL COMMENT '属性名称',                       # 属性名称
    PRIMARY KEY (`attr_key_id`)
)engine=InnoDB charset=utf8;
 
# 商品属性值
CREATE TABLE `item_attr_val` (
    `attr_key_id` INT(10) UNSIGNED NULL DEFAULT NULL COMMENT '属性ID',       # 对应 item_attr_key 表的 attr_key_id, 完成一对多关联
    `item_id` INT(10) UNSIGNED NULL DEFAULT NULL COMMENT '商品ID',           # 关联到商品
    `symbol` INT(10) NULL DEFAULT NULL COMMENT '属性编码',                   # 属性编码
    `attr_value` VARCHAR(255) NULL DEFAULT NULL COMMENT '属性值'             # 属性值
)engine=InnoDB charset=utf8;

```

插入数据后：

```
mysql> select * from item_attr_key;
+-------------+---------+-----------+
| attr_key_id | item_id | attr_name |
+-------------+---------+-----------+
|           1 |     128 | 颜色      |
|           2 |     128 | 尺码      |
+-------------+---------+-----------+
2 rows in set (0.02 sec)


mysql> select * from item_attr_val;
+-------------+---------+--------+------------+
| attr_key_id | item_id | symbol | attr_value |
+-------------+---------+--------+------------+
|           1 |     128 |      1 | 蓝色       |
|           1 |     128 |      2 | 白色       |
|           1 |     128 |      3 | 黑色       |
|           2 |     128 |      4 | S          |
|           2 |     128 |      5 | M          |
|           2 |     128 |      6 | L          |
+-------------+---------+--------+------------+
6 rows in set (0.00 sec)

```


symbol 字段是对指定商品 ID 下的属性值的一个序号标记, 是为了提高在后面使用到时的检索效率。该值在不同商品间可以重复, 在同一商品的属性中需要保证唯一。
以上就完成了商品 ID 为 128 的商品多属性的存储工作。
为了能够记录和快速查询出每种属性组合出的商品的价格、库存等信息, 我们还需要张表来维护这部分数据, 建立 item_sku 表:

```
CREATE TABLE `item_sku` (
    `sku_id` INT(10) UNSIGNED NOT NULL AUTO_INCREMENT COMMENT '主键 ID',                     
    `item_id` INT(10) UNSIGNED NOT NULL COMMENT '商品ID',                      
    `attr_symbol_path` VARCHAR(255) NOT NULL COMMENT '属性搭配方式',                              
    `price` DECIMAL(15,2) NOT NULL DEFAULT '0.00' COMMENT '价格',                          
    `stock` INT(10) UNSIGNED NOT NULL DEFAULT '0' COMMENT '库存数量',                          
    PRIMARY KEY (`sku_id`)
)engine=InnoDB charset=utf8;

```

将示例中具有三种颜色、三种尺码的属性数据生成 SKU 后的 item_sku 表数据图示:

```

mysql> select * from item_sku;
+--------+---------+------------------+--------+-------+
| sku_id | item_id | attr_symbol_path | price  | stock |
+--------+---------+------------------+--------+-------+
|      1 |     128 | 1,4              | 200.00 |   100 |
|      2 |     128 | 1,5              | 201.00 |   101 |
|      3 |     128 | 1,6              | 202.00 |   102 |
|      4 |     128 | 2,4              | 203.00 |   103 |
|      5 |     128 | 2,5              | 204.00 |   104 |
|      6 |     128 | 2,6              | 205.00 |   105 |
|      7 |     128 | 3,4              | 206.00 |   106 |
|      8 |     128 | 3,5              | 207.00 |   107 |
|      9 |     128 | 3,6              | 208.00 |   109 |
+--------+---------+------------------+--------+-------+
9 rows in set (0.02 sec)

```

从图中数据看出, 该商品共有 9 种不同规格可选, 那么这时在确定用户选择的某种规格的价格等信息时只需一条 SQL 语句即可完成:

```
mysql> select * from `item_sku` where `item_id`=128 and `attr_symbol_path`='1,4';
+--------+---------+------------------+--------+-------+
| sku_id | item_id | attr_symbol_path | price  | stock |
+--------+---------+------------------+--------+-------+
|      1 |     128 | 1,4              | 200.00 |   100 |
+--------+---------+------------------+--------+-------+
1 row in set (0.00 sec)

```

### 总结

如何属性规则再多，也可以利用上述方法，如果设计运费还可以在 SKU 表中加入运费字段。

