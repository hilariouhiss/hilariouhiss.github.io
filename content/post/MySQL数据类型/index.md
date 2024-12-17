---
title: MySQL数据类型
date: 2024-12-12
lastmod: 2024-12-17
categories:
  - 知识
  - MySQL
description: 对MySQL 数字类型，日期时间，字符串类型，空间数据类型和JSON类型的学习
---

## 分类

数值类型，日期和时间类型，字符串类型，空间数据类型，JSON类型

约定：

- *M*：对于整型，表示最大显示宽度；对于浮点型和顶点型，是可存储的总位数；对于字符串类型，是最大长度。*M* 的最大值取决与数据类型
- *D*：对于浮点型和定点型，表示小数点后的位数。其最大值可能是30，但不应该大于 *M* - 2
- *fsp*：对于 `TIME`，`DATETIME` 和 `TIMESTAMP`，表示秒的小数部分的位数；*fsp* 值的范围为 [0, 6]，默认为0，而非标准SQL的6，因为需要向后兼容
- `[]` 方括号表示可选部分
- ~~已弃用~~ 删除线表示已弃用，或被删除

## 数值类型

### 概述

[MySQL :: MySQL 8.0 Reference Manual :: 13.1.1 Numeric Data Type Syntax](https://dev.mysql.com/doc/refman/8.0/en/numeric-type-syntax.html)

在MySQL 8.0.17中，整型的显示宽度特性和 `ZEROFILL` 被标记为废弃。可使用 `LPAD()` 或将数字格式化存储为 `CHAR` 类型

整型默认为有符号，允许使用 `UNSIGNED` 变为无符号

对于 `FLOAT`，`DOUBLE` 和 `DECIMAL` 以及它们的别名，`UNSIGNED` 被标记为废弃，考虑使用 `CHECK` 来限制

`SERIAL` 是 `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE` 的别名

整数列定义中的 `SERIAL DEFAULT VALUE` 是` NOT NULL AUTO_INCREMENT UNIQUE` 的别名

**注意**：有无符号整数参与的减法的结果也是无符号的，除非开启 [`NO_UNSIGNED_SUBTRACTION`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_unsigned_subtraction) 模式

### 数据类型

| 数据类型                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `BIT[(M)]`                                                   | `M` 指定每个值的位数，范围为 `1` 到 `64`，默认为 `1`<br />`BIT[M]` 意思是 `M` 比特长度的值 |
| `TINYINT`                                                    | 1字节<br />有符号范围：`-128` 到 `127`                       |
| `BOOL`<br />`BOOLEAN`                                        | `TINYINT(1)` 的别名<br />`0` 表示 `false`，非零表示 `true`<br />`TRUE` 是 `1`，`FALSE` 是 `0` |
| `SMALLINT`                                                   | 2字节                                                        |
| `MEDIUMINT`                                                  | 3字节                                                        |
| `INT`<br />`INTEGER`                                         | 4字节                                                        |
| `BIGINT`                                                     | 8字节                                                        |
| `SERIAL`                                                     | `BIGINT UNSIGNED NOT NULL AUTO_INCREMENT UNIQUE` 的别名      |
| `FLOAT`                                                      | 单精度浮点数<br/>精度约为 7 位小数                           |
| `DOUBLE`                                                     | 双精度浮点数<br />精度约为 15 位小数                         |
| `FLOAT(p)`                                                   | 为兼容 ODBC 的语法<br />`p` 仅用于确定存储大小<br/>`0 ≤ p ≤ 24` 时，类型为 `FLOAT`<br />`25 ≤ p ≤ 53` 时，类型为 `DOUBLE` |
| `DECIMAL[(M[,D])]`                                           | 定点数<br />`M` 是总位数（最大 `65`），`D` 是小数位数（最大 `30`）<br/>默认值：`M=10, D=0`<br />可用此类型控制数字的精度 |
| `DEC[(M[,D])]`<br />`NUMERIC[(M[,D])]`<br />`FIXED[(M[,D])]` | `DECIMAL` 的同义词<br />`FIXED` 用于兼容其他数据库系统       |
| `DOUBLE PRECISION[(M,D)]`,<br />`REAL[(M,D)]`                | `DOUBLE` 的同义词。<br/>启用 `REAL_AS_FLOAT` SQL 模式时，`REAL` 是 `FLOAT` 的同义词 |

### 补充

#### 截断

当存储的数字的小数部分超过 `DECIMAL` 规定的长度，该数字会被转化为在合规的值，通常是被截断。

#### 可移植性

代码应该直接使用 `FLOAT` 和 `DOUBLE`，而不去指定精度或位数。

#### BIT(M)

若一个小于 `M` 位的值被赋予 `BIT(M)` 类型，则左侧添加 `0` 以补全，如 `101` 赋给 `BIT(6)`，则存储为 `000101**`**

#### AUTO_INCREMENT

支持整型，~~浮点型~~，不支持负数； `AUTO_INCREMENT` 序列从 `1` 开始；当 `NULL` 或 `0` 被插入 `AUTO_INCREMENT` 列时，序列自动加一，[`NO_AUTO_VALUE_ON_ZERO`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html#sqlmode_no_auto_value_on_zero) SQL模式被启用后，`0` 值失效；若列没有被定义为 `NOT NULL`，则会直接存储 `NULL`，而不是自动加一；当其他值如 `5` 被插入 `AUTO_INCREMENT` 列，则不做处理，并且序列被重置为从 `5` 开始。

#### 超限与溢出

##### 超限

`strict SQL` 模式中，当数据大小超限时，依照SQL标准，MySQL会拒绝数据，插入失败并抛出错误；非严格模式中，MySQL会将值限定在规定的范围之内，然后存储。例子如下：

```sql
CREATE TABLE t1 (i1 TINYINT, I2 TINYINT UNSIGNED);

mysql> SET sql_mode = '';
mysql> INSERT INTO t1 (i1, i2) VALUES(256, 256);
mysql> SHOW WARNINGS;
+---------+------+---------------------------------------------+
| Level   | Code | Message                                     |
+---------+------+---------------------------------------------+
| Warning | 1264 | Out of range value for column 'i1' at row 1 |
| Warning | 1264 | Out of range value for column 'i2' at row 1 |
+---------+------+---------------------------------------------+
mysql> SELECT * FROM t1;
+------+------+
| i1   | i2   |
+------+------+
|  127 |  255 |
+------+------+
```

##### 溢出

数字表达式结果溢出会导致错误，`UNSIGNED` 数据参与的减法中，若出现结果为负，则也会抛出错误，开启 `NO_UNSIGNED_SUBTRACTION` 模式后可行。例子如下：

```sql
mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT CAST(0 AS UNSIGNED) - 1;
ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(cast(0 as unsigned) - 1)'

mysql> SET sql_mode = 'NO_UNSIGNED_SUBTRACTION';
mysql> SELECT CAST(0 AS UNSIGNED) - 1;
+-------------------------+
| CAST(0 AS UNSIGNED) - 1 |
+-------------------------+
|                      -1 |
+-------------------------+
```

若使用负值更新一个 `UNSIGNED` 列，在启用 `NO_UNSIGNED_SUBTRACTION` 模式时，列的值会被置为 `0`，如果启用了 严格模式，MySQL将会报错。无论是否启用 `NO_UNSIGNED_SUBTRACTION`，超过上限的值都会被置为最大值。

## 日期和时间类型

[MySQL ：： MySQL 8.0 参考手册 ：： 13.2 日期和时间数据类型 --- MySQL :: MySQL 8.0 Reference Manual :: 13.2 Date and Time Data Types](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-types.html)

### 概述

数据类型分为：`DATE`，`TIME`，`DATETIME`，`TIMESTAMP`，`YEAR`，每个类型都具有各自的有效值范围和一个“零值”

`TIMESTAMP` 和 `DATETIME` 具有特殊的自动更新行为，[Automatic Initialization and Updating for TIMESTAMP and DATETIME”](https://dev.mysql.com/doc/refman/8.0/en/timestamp-initialization.html)

数据类型存储要求，[Data Type Storage Requirements](https://dev.mysql.com/doc/refman/8.0/en/storage-requirements.html)

日期和时间函数，[Date and Time Functions](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)

日期和时间字面量，[Date and Time Literals](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-literals.html)

输入与输出：MySQL 日期标准格式为“年-月-日”，其他格式可使用 `STR_TO_DATE()` 函数进行转换

两位数年份的解释，[2-Digit Years in Dates](https://dev.mysql.com/doc/refman/8.0/en/two-digit-years.html)：

- 70-99 被看作 1970-1999
- 00-69 被看作 2000-2069

类型之间的转换，[Conversion Between Date and Time Types](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-type-conversion.html)

当参与数值相关时，MySQL自动将日期时间的值转换为数值类型，反之亦然

无效值处理：默认情况下，当 MySQL 遇到超出范围或无效的日期或时间值时，它会将该值转换为该类型的“零”值。唯一的例外是超出范围的 `TIME` 值，它们会被截断为范围的一端，即最大值或最小值

MySQL允许存储 `day` 或 `month` 为零的 `DATE` 和 `DATETIME`，适用于存储不知道具体日期的值，但在如 `DATE_SUB()`或 `DATE_ADD()` 的这样要求完整日期的函数中，使用这样的值将可能不会得到正确的结果。若不允许可启用 `NO_ZERO_IN_DATE` 模式。

MySQL允许存储如 `0000-00-00` 全零值的日期，这样比使用 `NULL` 要方便一些，并且占用更少的存储和索引空间。若不允许可启用 `NO_ZERO_DATE` 模式。

通过 Connector/ODBC 使用的“零”日期或时间值会自动转换为 NULL，因为 ODBC 无法处理这样的值。

### 语法

`TIME`，`DATETIME`，`TIMESTAMP` 支持设置秒的小数精度，至高6位，格式如下：

```sql
CREATE TABLE t1 (t TIME(3), dt DATETIME(6), ts TIMESTAMP(0));  # 若不指定，默认为 0，与 SQL 标准的 6 不同
```

| 数据类型         | 描述                                                         | 零值                |
| ---------------- | ------------------------------------------------------------ | ------------------- |
| `DATE`           | 格式： `YYYY-MM-DD`<br />范围：'1000-01-01' 到 '9999-12-31'  | 0000-00-00          |
| `DATETIME(fsp)`  | 格式： `YYYY-MM-DD HH:MM:SS[.fraction]`<br />范围：'1000-01-01 00:00:00.000000' 到 '9999-12-31 23:59:59.499999' | 0000-00-00 00:00:00 |
| `TIMESTAMP(fsp)` | 格式与 `DATETIME` 类似，自动更新时区。<br />范围：'1970-01-01 00:00:01.000000' UTC 到 '2038-01-19 03:14:07.499999' UTC<br />存储的是自从 '1970-01-01 00:00:00' UTC 至今的秒数，所以无法表示 '1970-01-01 00:00:00' UTC，因为 `0` 表示的是 '0000-00-00 00:00:00' | 0000-00-00 00:00:00 |
| `TIME(fsp)`      | 格式：`HH:MM:SS[.fraction]`<br />范围：'-838:59:59.000000'` to `'838:59:59.000000' | 00:00:00            |
| `YEAR`           | 格式：`YYYY`<br />范围：1901 到 2155                         | 0000                |

均允许使用字符串或数值为其赋值

MySQL 服务器对 `TIMESTAMP` 列的处理方式依赖于 `explicit_defaults_for_timestamp` 系统变量的状态：

- **启用时**：所有 `TIMESTAMP` 行为都需要显式定义，包括默认值和自动更新规则，且未显式声明 `NOT NULL` 的 `TIMESTAMP` 列允许 `NULL` 值。
- **禁用时**：第一个 `TIMESTAMP` 列会自动处理插入和更新操作的时间戳，默认情况下具有 `DEFAULT CURRENT_TIMESTAMP` 和 `ON UPDATE CURRENT_TIMESTAMP` 属性，除非另有指定。此外，可以通过赋值 `NULL` 来更新 `TIMESTAMP` 列为当前时间戳，除非该列允许 `NULL` 值。

SUM() 和 AVG() 聚合函数不适用于时间值，因为它们会将值转换为数字，这将丢失第一个非数字字符之后的所有内容，解决方法示例：

```sql
SELECT SEC_TO_TIME(SUM(TIME_TO_SEC(time_col))) FROM tbl_name;
SELECT FROM_DAYS(SUM(TO_DAYS(date_col))) FROM tbl_name;
```

### DATE，DATETIME，TIMESTAMP

`TIMESTAMP`：MySQL 会将当前时间转为时区为 UTC 后再存储，查询时会再转为当前时区的时间。默认情况当前时区使用服务器的时区。可以为每个连接设置时区。

在 MySQL 8.0.19 及更高版本中，可以在将 `TIMESTAMP` 或 `DATETIME` 值插入表中时指定时区偏移量

在 MySQL 8.0.22 及更高版本中，在查询时，可以在使用带 `AT TIME ZONE` 运算符的 `CAST()` 函数将 `TIMESTAMP` 值转换为时区为 UTC 的 `DATETIME` 值，如下：

```sql
mysql> SELECT col,
     >     CAST(col AT TIME ZONE INTERVAL '+00:00' AS DATETIME) AS ut
     >     FROM ts ORDER BY id;
+---------------------+---------------------+
| col                 | ut                  |
+---------------------+---------------------+
| 2020-01-01 10:10:10 | 2020-01-01 15:10:10 |
| 2019-12-31 23:40:10 | 2020-01-01 04:40:10 |
| 2020-01-01 13:10:10 | 2020-01-01 18:10:10 |
| 2020-01-01 10:10:10 | 2020-01-01 15:10:10 |
| 2020-01-01 04:40:10 | 2020-01-01 09:40:10 |
| 2020-01-01 18:10:10 | 2020-01-01 23:10:10 |
+---------------------+---------------------+
```

### TIME

MySQL对于不带冒号的缩写，会将最右边的两个数看作“秒”，如 '11:12'，'1112'，1112，MySQL 均视为 '00:11:12'，而非 '11:12:00'。

秒的小数部分只能使用 `.` 分隔。

超出范围的值将被截断为最大或最小值，不合法的值将被置为“零值”。

### YEAR

插入数字 `0` 时，MySQL将存储和显示为 `0000`，插入字符 `'0'` 或 `'00'`，将被视为 `2000` 进行存储和显示

### TIMESTAMP 和 DATETIME 的自动初始化和自动更新

二者可以自动初始化和更新为当前时间

**自动初始化**：当插入新行，但该列未直接指定值时，值将自动置为当前时间戳，实现如下：

```sql
CREATE TABLE example (
    id INT AUTO_INCREMENT PRIMARY KEY,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**自动更新**：当同一行中的任何其他列的值改变时，该列将自动更新为当前的时间戳，新值与旧值完全相同则不会自动更新，实现如下：

```sql
CREATE TABLE example (
    id INT AUTO_INCREMENT PRIMARY KEY,
    updated_at ON UPDATE CURRENT_TIMESTAMP
);
```

- **防止自动更新**：如果不想在更新其他列时自动更新时间戳列，可以显式地将其设置为当前值。例如，在 `UPDATE` 语句中明确设置 `timestamp_col = timestamp_col`。
- **强制更新时间戳**：即使没有更新其他列，你也想更新时间戳列。这可以通过显式地将时间戳列设置为 `CURRENT_TIMESTAMP` 来实现。例如，在 `UPDATE` 语句中设置 `timestamp_col = CURRENT_TIMESTAMP`。

可自动初始化或更新的列，只能存在一列。

如果显式地指定了秒的精度，则 `DEFAULT CURRENT_TIMESTAMP` 和 `ON UPDATE CURRENT_TIMESTAMP` 也必须显式指定相同的秒的精度。

```sql
CREATE TABLE t1 (
  ts TIMESTAMP(6) DEFAULT CURRENT_TIMESTAMP(6) ON UPDATE CURRENT_TIMESTAMP(6)
);
```

当系统变量 `explicit_defaults_for_timestamp` 被禁用，可以为 `NOT NULL` 的 `TIMESTAMP` 列使用 `NULL` 来实现自动初始化和更新。

`DEFAULT CURRENT_TIMESTAMP` 和 `ON UPDATE CURRENT_TIMESTAMP` 没有顺序之分。

 `CURRENT_TIMESTAMP`，`CURRENT_TIMESTAMP()`，`NOW()`，`LOCALTIME`，`LOCALTIME()`，`LOCALTIMESTAMP`，`LOCALTIMESTAMP()` 互等。

在如下情况中，列的默认值取决于其数据类型：

```sql
CREATE TABLE t1 (
  ts1 TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,     -- 默认值 0
  ts2 TIMESTAMP NULL ON UPDATE CURRENT_TIMESTAMP -- 默认值 NULL
);

-- DATETIME 的默认值为 NULL，在列被定义为 NOT NULL 时，默认值变为 0
```

### TIME 中秒的精度

当如下舍入情况发生时，MySQL 将不会有任何警告或错误：

```sql
CREATE TABLE fractest( c1 TIME(2), c2 DATETIME(2), c3 TIMESTAMP(2) );
INSERT INTO fractest VALUES
('17:51:04.777', '2018-09-08 17:51:04.777', '2018-09-08 17:51:04.777');

mysql> SELECT * FROM fractest;
+-------------+------------------------+------------------------+
| c1          | c2                     | c3                     |
+-------------+------------------------+------------------------+
| 17:51:04.78 | 2018-09-08 17:51:04.78 | 2018-09-08 17:51:04.78 |
+-------------+------------------------+------------------------+
```

可通过开启 `TIME_TRUNCATE_FRACTIONAL` 模式来变为直接截断，而非舍入。

### 各类型之间的转换

`DATE`：

- 转为 `DATETIME` 和 `TIMESTAMP` 时，时间部分被置为 `00:00:00`
- 转为 `TIME`，结果为 `00:00:00`

`DATETIME` 和 `TIMESTAMP`：

- 转为 `DATE` 时，时间部分会进行舍入，如 1999-12-31 23:59:59.499 变为 1999-12-31，1999-12-31 23:59:59.500 变为 2000-01-01
- 转为 `TIME` 时，日期部分被丢弃

`TIME`：当转为其他带日期的类型时，会使用 `CURRENT_DATE()` 作为日期部分，并且时间被视为已过去的时间。假设当前日期为 2012-01-01。`TIME` 的值分别为 12:00:00、24:00:00 和 -12:00:00，在转换为 `DATETIME` 或 `TIMESTAMP` ，分别生成 2012-01-01 12:00:00、2012-01-02 00:00:00 和 2011-12-31 12:00:00。

### 两位数的 YEAR

`ORDER BY` 将按值的大小对其进行排序

一些如 `MIN()` 和 `MAX()` 的函数使用这种 `YEAR` 无法输出正确的结果，最好补为 4 位。

## 字符串类型

| 数据类型                       | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `CHAR[(M)]`                    | 定长字符串<br />M：0 ~ 255，默认为 1<br />查询时默认移除尾部的空格，除非启用`PAD_CHAR_TO_FULL_LENGTH` 模式 |
| `VARCHAR(M)`                   | 可变长度字符串，使用 1 或 2 字节存储长度前缀，超过 255 字节使用 2 字节<br />最大长度受行大小（65,535 字节）和字符集影响 |
| `BINARY[(M)]`                  | 存储二进制字节字符串<br />M 默认为 1 字节                    |
| `VARBINARY(M)`                 | 同上                                                         |
| `TINYBLOB`<br />`TINYTEXT`     | 最长 255 字节<br />使用 1 字节前缀指明总字节数               |
| `BLOB[(M)]`<br />`TEXT`        | 最长 65,535 字节<br />使用 2 字节前缀指明总字节数            |
| `MEDIUNBLOB`<br />`MEDIUMTEXT` | 最长 16,777,215 字节<br />使用 3 字节前缀指明总字节数        |
| `LONGBLOB`<br />`LONGTEXT`     | 最长 4,294,967,295 字节，4GB<br />使用 4 字节前缀指明总字节数 |
| `ENUM('value1','value2',...)`  | 枚举类型<br />只能含有值列表中的一个值，或 `NULL`，或代表错误的 `''` 的字符串对象<br />值在内部表示为整型<br />一列最多可有 65,535 个不同元素<br />单独的枚举元素需满足：M <= 255 且 (M × w) <= 1020，M 为元素长度，w 为对应编码占用最大字节数 |
| `SET('value1','value2',...)`   | 集合类型<br />存储零个或多个存在于值列表中的值的字符串对象<br />值在内部表示为整型<br />一列可有至多 64 个不同成员<br />单个成员需满足：M <= 255 且 (M × w) <= 1020，M 为元素长度，w 为对应编码占用最大字节数 |

对于 `CHAR`，`VARCHAR`，`TEXT` 类型，长度以“字符”为单位，对于 `BINARY`，`VARBINARY`，`BLOB` 类型，长度以“字节”为单位

`CHAR`，`VARCHAR`，`TEXT`，`ENUM`，`SET` 以及它们的同义词，都可使用 `CHARACTER SET` 和 `COLLATE` 来分别设置编码和排序规则

`CHARSET` 等同于 `CHARACTER SET`

若对 `CHAR`，`VARCHAR`，`TEXT` 设置其编码为 `binary`，MySQL 会自动进行如下转换：

```sql
CREATE TABLE t
(
  c1 CHAR(10) CHARACTER SET binary,
  c2 VARCHAR(10) CHARACTER SET binary,
  c3 TEXT CHARACTER SET binary,
);
-- 转为如下
CREATE TABLE t
(
  c1 BINARY(10),
  c2 VARBINARY(10),
  c3 BLOB,
);
-- 对于 ENUM 和 SET 则不进行
```

~~`BINARY` 还可被看作是使用对应编码的二进制排序规则的简写，示例如下：~~

```sql
CREATE TABLE t
(
  c1 VARCHAR(10) CHARACTER SET latin1 BINARY,
  c2 TEXT BINARY
) CHARACTER SET utf8mb4;
-- 定义如下：
CREATE TABLE t (
  c1 VARCHAR(10) CHARACTER SET latin1 COLLATE latin1_bin,
  c2 TEXT CHARACTER SET utf8mb4 COLLATE utf8mb4_bin
) CHARACTER SET utf8mb4;
```

`ASCII` 是 `CHARACTER SET latin1` 的简写，`UNICODE` 是  `CHARACTER SET ucs2` 的简写，但二者在MySQL 8.0.28 及以后版本中已被废弃。

对于 `CHAR`，`VARCHAR`，`TEXT`，`ENUM` 和 `SET`，使用 `_bin` 排序规则以使用底层字符编码值而非词汇顺序进行排序

MySQL允许创建 `CHAR(0)` 列，对于适配必须有此列，但不会使用此列数据的应用十分有用，这种方式只占用 1 bit，只能取 `NULL` 和 `''`。

### CHAR，VARCHAR

`CHAR` 存储固定长度的字符串，长度不够者右侧用空格补齐，查询时去除。

对于 `VARCHAR`，若数据尾随空格超长，MySQL将自动截断并**给出警告**，若非空格字符超长则报错，关闭严格SQL模式后将截断并生成警告。

对于 `CHAR`，若数据尾随空格超长，MySQL将自动截断，**并不会警告**，但非空格字符超长则报错，关闭严格SQL模式后将截断并生成警告。

加上 `TEXT`，三者在比较排序时，使用 `NO PAD` 的排序规则将视尾随空格与其他字符为相同地位，`PAD SPACE` 的排序规则将无视尾随空格。

### BINARY，VARBINARY

超长数据会导致报错，关闭严格SQL模式后截断存储并警告

对于 `BINARY`，不够固定长度的数据会在右侧补 `0x00`（即 `\0`）以补齐长度，并且查询时不去除，`0x00` 是零字节，顺序在空格前

对于 `VARBINARY`，不会填充补齐，查询时也无去除操作，且所有数据在比较操作中都地位相同，如 `ORDER BY` 和 `DISTINCT`

若一 `BINARY` 列有唯一值索引，则在有 `'a'` 的情况下插入 `'a '` 会造成键重复错误。

MySQL客户端中，二进制字符串使用十六进制显示，使用 `--binary-as-hex` 参数修改

### BLOB，TEXT

二者均有小，标准，中，大四种子类型，且一一对应，只是存储的数据类型不同

对于非尾随空格超长的数据均会报错，未启用严格SQL模式时会自动截断并警告

尾随空格超长的数据插入到 `TEXT` 中时，空格会被截断并警告，不管SQL模式

插入时均无填充补齐，查询时也无任何去除

当 `TEXT` 列被索引时，索引条目在比较时会在末尾用空格补齐，如在有 `'a'` 的情况下插入 `'a '` 将引发键重复错误。`BLOB` 则不会。

在大多数情况，`TEXT` 可被看作极大的 `VARCHAR`，`BLOB` 可被看作极大 `VARBINARY`，但也有如下区别：

- 当在 `BLOB` 和 `TEXT` 列建立索引时，必须指定索引前缀的长度，对于 `VARCHAR` 和 `VARBINARY` 则是无所谓的。
- `BLOB` 和 `TEXT` 无法使用 `DEFAULT` 来设置默认值。

`LONG` 和 `LONG VARCHAR` 映射到 `MEDIUMTEXT`，兼容之举。

Connector/ODBC 将 `BLOB` 值定义为 `LONGVARBINARY`，将 `TEXT` 值定义为 `LONGVARCHAR`。

因为二者可以非常长，所以使用时有以下限制：

- 排序或分组时只使用前 `max_sort_length` 个字节的数据，`max_sort_length` 默认值为 1024，修改：`SET max_sort_length = 2000;`
- 查询含有二者的表，若有临时表参与，服务端将使用磁盘上的表而非内存中的，因为内存引擎不支持这两种数据类型，所以在非必要情况下不要让查询结果包含二者。
- `BLOB` 或 `TEXT` 的最大大小由其类型确定，但实际上，客户端和服务器之间传输的最大值由可用内存量和通信缓冲区的大小决定。
- `BLOB` 或 `TEXT` 每次被读取时都会分配一次内存，而其他数据类型都只在打开表时分配一次内存。

### ENUM

优势：

- 缩减取值个数有限的列的存储空间。因为输入的字符串被编码为数字。
- 可读性。查询时数字又被显示为对应的字符串。

注意事项：

- 若是枚举值与数字十分相似，如 `'1'`，`'2'`之类的值，则容易与它们对应的编码值混淆
- 使用 `ORDER BY` 需注意

使用如下：

```sql
CREATE TABLE shirts (
    name VARCHAR(40),
    size ENUM('x-small', 'small', 'medium', 'large', 'x-large')
);
INSERT INTO shirts (name, size) VALUES ('dress shirt','large'), ('t-shirt','medium'),
  ('polo shirt','small');
SELECT name, size FROM shirts WHERE size = 'medium';
+---------+--------+
| name    | size   |
+---------+--------+
| t-shirt | medium |
+---------+--------+
UPDATE shirts SET size = 'small' WHERE size = 'large';
COMMIT;
```

列表中的枚举值对应的编码值从 1 开始，而 `NULL` 对应 `NULL`，`''` 对应 `0`（表示出错，如未启用严格SQL模式下被赋予不合法的值时），可使用 `SELECT * FROM tbl_name WHERE enum_col=0;` 来查看是否有行被赋予非法值

若列允许 `NULL`，则 `NULL` 是默认值，若列为 `NOT NULL`，则默认值是枚举值列表的第一个。

可使用 `SELECT enum_col+0 FROM tbl_name;` 来使结果显示为编码值

表创建时，值列表成员的尾随空格被自动删除

查询时，显示的枚举值格式与表定义中的格式相同，而非插入时的格式，如下：

```sql
CREATE TABLE example (
    status ENUM('Active', 'Inactive', 'Pending')
);
INSERT INTO example (status) VALUES ('ACTIVE'), ('inactive'), ('PENDING');
SELECT status FROM example;
+----------+
| status   |
+----------+
| Active   |
| Inactive |
| Pending  |
+----------+
```

如果一个数字以字符形式出现，如 `'3'`，若枚举值列表中没有 `'3'`，则它将被视为数字 3，即编码值，这时将存储编码 3 对应枚举值。

排序时，`NULL` 永远是第一个，`''` 排第二，其余按编码值排序

使用 `ORDER BY CAST(col AS CHAR)` 或 `ORDER BY CONCAT(col)` 让枚举值按词汇顺序排序

枚举值不能是一个表达式，即使表达式结果是单个字符串，也无法是一个用户变量，示例如下：

```sql
CREATE TABLE sizes (
    size ENUM('small', CONCAT('med','ium'), 'large')
);
-- 错误
SET @mysize = 'medium';

CREATE TABLE sizes (
    size ENUM('small', @mysize, 'large')
);
-- 错误
```

### SET

集合类型，成员之间使用 `,` 隔开

严格模式启用时，表定义时若有重复的值出现会造成警告或错误

表创建时，集合成员的尾随空格将被自动去除

查询时，集合成员以定义时的格式显示，而非插入时的格式

集合中的成员被编码为“位标志”，如 `SET('a','b','c','d')` 中，成员编码如下：

| `SET` 成员 | 十进制值 | 二进制值 |
| :--------- | :------- | :------- |
| `'a'`      | `1`      | `0001`   |
| `'b'`      | `2`      | `0010`   |
| `'c'`      | `4`      | `0100`   |
| `'d'`      | `8`      | `1000`   |

如果赋值 9，即二进制 1001，MySQL 将选择第一个和第四个成员 `a` 和 `d`，结果为`'a,d'`。

插入时，集合成员出现的顺序和次数均不重要，查询时将按定义时的顺序排序，且每个成员只出现一次。

若插入时值存在不合法的成员，MySQL将报错并阻止插入，若关闭严格模式，MySQL将告警并只将合法的成员插入

通常使用 `FIND_IN_SET()` 函数或 `LIKE` 运算符来查找集合中的成员，示例如下：

```sql
mysql> SELECT * FROM tbl_name WHERE FIND_IN_SET('value',set_col)>0;
-- 查找 set_col 列是否存在 value 成员
-- FIND_IN_SET 函数是将列按 , 分隔，然后依次匹配，要求列必须是 SET 或 按 , 分隔的列表。返回 value 所在的位置，首位为 1，失败为 0

mysql> SELECT * FROM tbl_name WHERE set_col LIKE '%value%';
-- 在 set_col 列直接进行模式匹配，a,ab,c 中查找 b 也可成功，不精确。成功返回 1，失败则 0
```

也可使用 `=` 或 逻辑与 `&` 进行判断，示例如下：

```sql
mysql> SELECT * FROM tbl_name WHERE set_col & 1;

mysql> SELECT * FROM tbl_name WHERE set_col = 'val1,val2';
-- 查询结果与 'val2,val1' 不同，注意按定义时的顺序
```

使用 `SHOW COLUMNS FROM tbl_name LIKE 'set_col';` 来查看表中 `set_col` 集合列的所有可能成员。

## 空间数据

暂略

[MySQL :: MySQL 8.0 Reference Manual :: 13.4 Spatial Data Types](https://dev.mysql.com/doc/refman/8.0/en/spatial-types.html)

## JSON

[MySQL :: MySQL 8.0 Reference Manual :: 13.5 The JSON Data Type](https://dev.mysql.com/doc/refman/8.0/en/json.html)

MySQL 遵从 [RFC 7159](https://tools.ietf.org/html/rfc7159) 定义的 JSON

优势：

- 自动验证存储在其中的JSON数据
- 存储格式优化。增加了读取的速度；服务器读取时无需先转为文本再解析；使用结构化的二进制格式存储，使得服务器可依照键或索引查找，而无需读入整个JSON。

MySQL 8 支持 ***JSON Merge Patch*** 格式，通过 `JSON_MERGE_PATCH()` 函数来使用。

暂略
