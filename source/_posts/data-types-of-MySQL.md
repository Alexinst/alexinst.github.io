---
title: 「SQL」3. MySQL 的数据类型
date: 2019-06-06 15:44:34
tags:
	- SQL
	- MySQL
	- data types
categories:
	- SQL
---

**简述**：`MySQL`的数据类型可以分为三类：文本、数字和日期/时间类型。

<!-- more -->

<br />



# 1 数据类型

在 `MySQL` 中，有三种主要的类型：文本、数字和日期/时间类型。



## 1.1 Text 类型

| 数据类型              | 描述                                                         |
| :-------------------- | :----------------------------------------------------------- |
| `CHAR(size)`          | 固定长度（可包含字母、数字以及特殊字符），`size`指字符串的长度，最大为 255。 |
| `VARCHAR(size)`       | 可变长度。`size`指字符串的长度，最大为 255。如果大于 255，则被转换为`TEXT`类型。 |
| `TINYTEXT`            | 可变长度，最多 255 个字符。                                  |
| `TEXT`                | 可变长度，最多 65,535 个字符。                               |
| `MEDIUMTEXT`          | 可变长度，最多 16,777,215 个字符。                           |
| `LONGTEXT`            | 可变长度，最多 4,294,967,295 个字符。                        |
| `BLOB`                | `BLOBs(Binary Large Objects)`，存放最多 65,535 `bytes`的数据。 |
| `MEDIUMBLOB`          | 存放最多 16,777,215 `bytes`的数据。                          |
| `LONGBLOB`            | 存放最多 4,294,967,295 `bytes`的数据。                       |
| [ENUM(x,y,z,etc.)][1] | 枚举列表，可在 `ENUM` 列表中列出最大 65535 个值（字符串）。如果列表中不存在插入的值，则插入空值。<br>**注释**：这些值是按照输入的顺序存储的。格式：`ENUM('X','Y','Z')` |

> An `ENUM` is a string object with a value chosen from a list of permitted values that are enumerated explicitly in the column specification at table creation time.

[1]: https://www.yiibai.com/mysql/enum.html



## 1.2 整型类型

| 数据类型    | 字节数 | 有符号                                      | 无符号                    |
| :---------- | :----: | :------------------------------------------ | ------------------------- |
| `TINYINT`   |   1    | -128 ～127                                  | 0 ～ 255                  |
| `SMALLINT`  |   2    | -32768 ～ 32767                             | 0 ～ 65535                |
| `MEDIUMINT` |   3    | -8388608 ～ 8388607                         | 0 ～ 16777215             |
| `INT`       |   4    | -2147483648 ～ 2147483647                   | 0 ～ 4294967295           |
| `BIGINT`    |   8    | -9223372036854775808 ～ 9223372036854775807 | 0 ～ 18446744073709551615 |

整数类型拥有额外的选项 `UNSIGNED`。



## 1.3 浮点数类型

| 数据类型       | 字节数 | 描述                                                         |
| :------------- | :----: | :----------------------------------------------------------- |
| `FLOAT(M,D)`   |   4    | `M` 代表显示长度，`D` 代表小数位数，这两个参数都不是必需参数，默认为10, 2，小数精度可以达到24位 |
| `DOUBLE(M,D)`  |   8    | `M`和`D`默认为16, 4，小数精度可以达到53位。                  |
| `DECIMAL(M,D)` |        | 作为字符串存储，是非压缩的无符号浮点数。 每一位十进制数都对应一个字节。 |

`DECIMAL` 和 `FLOAT`/`DOUBLE`的区别，主要有两点：

- `FLOAT`/`DOUBLE`在`db`中存储的是近似值，而`DECIMAL` 则是以字符串形式进行保存的；
- `DECIMAL(M,D)`的规则和`FLOAT`/`DOUBLE`相同，但`FLOAT`/`DOUBLE`在不指定M、D时默认按照实际精度来处理,而`DECIMAL` 在不指定M、D时默认为`DECIMAL (10, 0)`。



## 1.4 Date 类型

| 数据类型      | 字节数 |        格式         | 描述                                                         |
| :------------ | :----: | :-----------------: | ------------------------------------------------------------ |
| `DATE()`      |   3    |     YYYY-MM-DD      | 支持的范围是  1000-01-01 ~ 9999-12-31                        |
| `TIME()`      |   3    |      HH:MM:SS       | 支持的范围是  -838:59:59 ~ 838:59:59                         |
| `YEAR()`      |   1    |      YY / YYYY      | 2 /4 位格式的年份。4 位：1901 ～ 2155；2 位：70 ～ 69（1970 ～2069）。 |
| `DATETIME()`  |   8    | YYYY-MM-DD HH:MM:SS | 范围是 1000-01-01 00:00:00 ~ 9999-12-31 23:59:59             |
| `TIMESTAMP()` |   4    | YYYY-MM-DD HH:MM:SS | 时间戳。`TIMESTAMP` 值使用 `Unix` 纪元(1970-01-01 00:00:00 UTC) 至今的描述来存储。范围是 1970-01-01 00:00:01 UTC 到 2038-01-09 03:14:07 UTC |

`DATETIME` 和 `TIMESTAMP`两种类型的区别：

- `DATETIME` 与`TIMESTAMP`能存储的时间范围也不同，`DATETIME` 的存储范围为1000-01-01 00:00:00 ~ 9999-12-31 23:59:59，`TIMESTAMP`存储的时间范围为19700101080001 ~ 20380119111407
- `DATETIME`默认值为空；`TIMESTAMP`默认值不为空，当插入值为`null`时，`MySQL`会取当前时间
- `DATETIME` 存储的时间与时区无关，`TIMESTAMP`存储的时间及显示的时间都依赖于当前时区
- `TIMESTAMP` 也接受不同的格式，比如 
  - YYYY-MM-DD HH:MM:SS
  - YY-MM-DD HH:MM:SS
  - YYYY-MM-DD 
  - YY-MM-DD

<br />



# 2 系列

1. {% post_link installation-and-simple-instructions-of-sql 「SQL」1. 安装 MySQL %}
2. {% post_link basic-instructions-of-sql 「SQL」2. SQL格式规范、基本命令与范例 %} 
3. {% post_link data-types-of-MySQL 「SQL」3. MySQL 的数据类型  %}



<br />

**以上！**

<br />

