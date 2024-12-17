---
title: MySQL查询
date: 2024-12-16 14:39:17
categories:
  - 知识
  - MySQL
description: 内连接，左外连接，右外连接，全外连接，交叉连接，自连接，Union，函数等......
---

## 表连接

### 内连接

INNER JOIN，返回两个表中满足连接条件的记录



### 左外连接

LEFT JOIN，返回左表中的所有记录，右表中没有的记录设为 `NULL`



### 右外连接

RIGHT JOIN，返回右表中的所有记录，左表中没有的记录设为 `NULL`



### 全外连接

FULL JOIN，返回两个表中所有记录，当某一行在另一张表中没有匹配的记录时，设为 `NULL`
MySQL 不直接支持，需要通过 UNION 和左右外连接结合实现



### 交叉连接

CROSS JOIN，返回两表的笛卡尔积，即第一张表每一行与第二张表每一行的组合。
结果集的行数 = 两表行数的乘积
若不带 ON 子句，则是简单的笛卡尔积，若带有，则等价于 内连接。

### 自连接

SELF JOIN，表自己与自己连接
常用于层次结构数据

### Union

用于合并多个 `Select` 语句的结果集的操作符，用法如下：

```sql
SELECT column1, column2, ...
FROM table1
UNION [ALL]
SELECT column1, column2, ...
FROM table2
UNION [ALL]
SELECT column1, column2, ...
FROM table3
[ORDER BY column1];
```

`Union` 会去重，影响性能，`Union All` 则不会，非必要时考虑使用后者。

使用 `Union` 的要求：

1. 结果集的列数必须相同
2. 对应列的数据类型必须兼容，否则MySQL可能进行隐式转换，导致非预期结果出现
3. 合并后的结果集列名取自第一个 `Select` 语句
4. `Order By` 只能放在最后，意为对合并后的结果集进行排序

常用情景：

1. 合并不同来源的同格式数据
2. 生成报表
3. 一同处理历史数据和当前数据

## 函数

### count()

| 用法            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| count(*)        | MySQL 8 将优化为 count(常量)，理论上与 count(1) 性能相同。<br />不同的存储引擎可能有其他操作 |
| count(1)        | 扫描全表，统计有多少行                                       |
| count(col_name) | 剔除列值为 `NULL` 的行，剩余的总行数<br />添加 `Distinct` 还可去重，但性能更差 |
