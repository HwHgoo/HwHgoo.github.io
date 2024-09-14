---
title: Mysql Explain之type详解
date: 2024-09-12 10:14:32
Categories:
    - MySQL
tags:
    - Index
    - Explain
---

# Mysql Explain中的type字段详解

## 可能的值

按好坏排序：
`system, const, eq_ref, ref, fulltext, ref_or_null, index_merge, unique_subquery, index_subquery, range, index, ALL`

下面依次介绍各个值的含义，并对常见的做出着重的说明。

## system
在表中仅有一条数据时，mysql优化器会选择这种类型。
例如:

```sql
CREATE TABLE `only_one_row` (
  `id` int NOT NULL AUTO_INCREMENT,
  `col1` varchar(255) COLLATE utf8mb4_0900_as_cs DEFAULT NULL,
  `col2` varchar(255) COLLATE utf8mb4_0900_as_cs DEFAULT NULL,
  `col3` varchar(255) COLLATE utf8mb4_0900_as_cs DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_as_cs;

INSERT INTO only_one_row (col1, col2, col3) VALUES ('value1', 'value2', 'value3');

explain select * from only_one_row;
```
![](only-one-row.png)

<u>注意到`only_one_row`表的引擎为`MyISM`, 在使用`InnoDB`时，同样的查询不会选择`system`，而是`ALL`</u>

## const
查询最多只有一行数据匹配。当查询条件使用主键或唯一索引的<u>**所有**</u>部分，且与常数值匹配时，被查询的表会被用作 `const` 表。
例如：

```sql
    CREATE TABLE `tbl_const_example` (
        `id` INT NOT NULL AUTO_INCREMENT,
        `col1` INT NOT NULL,
        `col2` INT NOT NULL,
        `col3` VARCHAR(255) NOT NULL,
        PRIMARY KEY (`id`),
        UNIQUE KEY `uk` (`col2`, `col3`)
    ) ENGINE=InnoDB;

    INSERT INTO tbl_const_example (col1, col2) VALUES (1, 2, 'value1');
    INSERT INTO tbl_const_example (col1, col2) VALUES (2, 3, 'value2');
    INSERT INTO tbl_const_example (col1, col2) VALUES (3, 4, 'value3');

    EXPLAIN SELECT * FROM tbl_const_example WHERE `id` = 1; -- use const table
    EXPLAIN SELECT * FROM tbl_const_example WHERE `col2` = 2 AND `col3` = 'value2'; -- use const table
    EXPLAIN SELECT * FROM tbl_const_example WHERE `col2` = 2 AND `col3` = 'value2' AND `col1` = 1; -- use const table

    EXPLAIN SELECT * FROM TBL_CONST_EXAMPLE WHERE `col2` = 2; -- not use const table
```
![](const-example.png)

## eq_ref

```sql
CREATE TABLE `ref_tb1` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `col1` INT NOT NULL,
    `col2` INT NOT NULL,
    `col3` INT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk` (`col1`),
) ENGINE=InnoDB;

CREATE TABLE `ref_tb2` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `col1` INT NOT NULL,
    `col2` INT NOT NULL,
    `col3` INT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk` (`col2`)
) ENGINE=InnoDB;

CREATE TABLE `ref_tb3` (
    `id` INT NOT NULL AUTO_INCREMENT,
    `col1` INT NOT NULL,
    `col2` INT NOT NULL,
    `col3` INT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `uk` (`col3`)
) ENGINE=InnoDB;

INSERT INTO ref_tb1 (col1, col2, col3) VALUES (1, 2, 3), (4, 5, 6), (7, 8, 9);
INSERT INTO ref_tb2 (col1, col2, col3) VALUES (1, 2, 3), (4, 5, 6), (7, 8, 9);
INSERT INTO ref_tb3 (col1, col2, col3) VALUES (1, 2, 3), (4, 5, 6), (7, 8, 9);

EXPLAIN SELECT * FROM ref_tb1, ref_tb2 WHERE ref_tb1.col1 = ref_tb2.col2; -- use eq_ref
EXPLAIN SELECT * FROM ref_tb1, ref_tb2 WHERE ref_tb1.id = ref_tb2.id; -- use eq_ref
EXPLAIN SELECT * FROM ref_tb1, ref_tb2 WHERE ref_tb1.id = ref_tb2.col2; -- use eq_ref
EXPLAIN SELECT * FROM ref_tb1, ref_tb2 WHERE ref_tb1.id = ref_tb3.col3; --  use eq_ref


```
![](eq-ref.png)
