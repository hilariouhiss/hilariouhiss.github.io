---
title: MySQL查询
date: 2024-12-16
slug: mysql-query
categories:
  - MySQL
description: 内连接，左外连接，右外连接，全外连接，交叉连接，自连接，Union，函数等......
draft: true
lastmod: 2024-12-18
---
## 语句执行顺序

1. `FROM` 和 `JOIN`：首先，`FROM` 子句中的表被读取，若有连接则进行连接
2. `ON`：表连接时，会根据 `ON` 中指定的条件，搜索满足条件的行进行连接
3. `WHERE`：在表连接后，根据 `WHERE` 中的条件对连接后的表进行筛选
4. `GROUP BY`：筛选完成后，按照 `GROUP BY` 指定的列进行分组
5. 聚合函数：分组完成后，聚合函数将分别在每个分组中执行
6. `SELECT`：确定最终会被输出的列，在这个阶段，所有表达式都会被计算
7. `HAVING`：过滤不满足条件的分组
8. `ORDER BY`：对最终结果进行排序

注意：以上执行顺序只是逻辑上的执行顺序，真正的顺序由数据库优化后确定

`HAVING` 和 `WHERE` 的区别:
- `HAVING` 是对表分组后的结果进行过滤
- `WHERE` 是对表连接后的原始行进行筛选
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

例题：[1280. 学生们参加各科测试的次数 - 力扣(LeetCode)](https://leetcode.cn/problems/students-and-examinations/description/?envType=study-plan-v2&envId=sql-free-50)

学生表
```
Students
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| student_id    | int     |
| student_name  | varchar |
+---------------+---------+
PK: student_id
```
科目表：
```
Subjects
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| subject_name | varchar |
+--------------+---------+
PK: subject_name
```
考试表：

```
Examinations
+--------------+---------+
| Column Name  | Type    |
+--------------+---------+
| student_id   | int     |
| subject_name | varchar |
+--------------+---------+
PK: null
```

查询每个学生参加每一门科目测试的次数，结果按 `student_id` 和 `subject_name` 排序。
答案(非官方)
```sql
SELECT 
    stu.student_id,
    student_name,
    sub.subject_name,
    COUNT(e.subject_name) AS attended_exams
FROM
    students stu
        CROSS JOIN
    subjects sub
        LEFT JOIN
    examinations e USING (student_id ，subject_name)
GROUP BY stu.student_id ，sub.subject_name
ORDER BY stu.student_id ，sub.subject_name;
```

思路：
1. 每一个学生在每一门科目的测试次数，说明即使一名学生没有参加过一门科目的考试，也需要在结果集中显示为 0，可通过 `students cross join subjects` 得到这样的中间结果集
2. 中间结果再左连接 `examinations` 条件为 `using(student_id，subject_name)`，如此得到所有学生在每个科目的考试信息，若有学生没有参加过某一门科目的考试，则为 `null`
3. 在根据学生和科目进行分组，`count(e.subject_name)` 不计算 `null` 行，最后排序
### 自连接

SELF JOIN，表自己与自己连接
常用于层次结构数据

[570. 至少有5名直接下属的经理 - 力扣(LeetCode)](https://leetcode.cn/problems/managers-with-at-least-5-direct-reports/description/?envType=study-plan-v2&envId=sql-free-50)

```sql
SELECT 
    Manager.Name AS Name
FROM
    Employee AS Manager
        JOIN
    Employee AS Report ON Manager.Id = Report.ManagerId
GROUP BY Manager.Id
HAVING COUNT(Report.Id) >= 5
```

### Union

用于合并多个 `Select` 语句的结果集的操作符，用法如下：

```sql
SELECT column1，column2，...
FROM table1
UNION [ALL]
SELECT column1，column2，...
FROM table2
UNION [ALL]
SELECT column1，column2，...
FROM table3
[ORDER BY column1];
```

`Union` 会去重，影响性能，`Union All` 则不会，非必要时考虑使用后者。

使用 `Union` 的要求：

1. 结果集的列数必须相同
2. 对应列的数据类型必须兼容，否则 MySQL 可能进行隐式转换，导致非预期结果出现
3. 合并后的结果集列名取自第一个 `Select` 语句
4. `Order By` 只能放在最后，意为对合并后的结果集进行排序

常用情景：

1. 合并不同来源的同格式数据
2. 生成报表
3. 一同处理历史数据和当前数据

## 函数

### COUNT

| 用法            | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| COUNT(*)        | MySQL 8 将优化为 count(常量)，理论上与 count(1) 性能相同。<br />不同的存储引擎可能有其他操作 |
| COUNT(1)        | 扫描全表，统计有多少行                                       |
| COUNT(col_name) | 剔除列值为 `NULL` 的行，剩余的总行数<br />添加 `Distinct` 还可去重，但性能更差 |
### AVG

`AVG()` 通常用于计算平均值，传入列名表示计算此列的平均值

`AVG()` 还可以传入表达式，例如 `AVG()` 

### SUM

`SUM()` 与 `AVG()` 类似，既可以传入列名，也可以传入表达式

`SUM(price * count)`