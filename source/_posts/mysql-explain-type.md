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

## const