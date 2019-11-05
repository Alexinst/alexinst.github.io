---
title: 「SQL」2. SQL格式规范、基本命令与范例
date: 2019-05-08 19:56:57
tags:
	- SQL
	- MySQL
	- instructions
categories:
	- SQL
---

**简述**：这里只记录`SQL`的部分基本指令，包括`SELECT FROM`，`WHERE`，`GROUP BY`，`ORDER BY`。

<!-- more -->

<br />

# SQL要点

## 注释

`SQL`语句中的单行注释使用`--`，多行注释采用` /*…*/`

## 代码规范

   [SQL编程格式的优化建议](https://zhuanlan.zhihu.com/p/27466166)
   [SQL Style Guide](https://www.dsqlstyle.guide/)

## 执行顺序

`SQL` 语句有一个让大部分人都感到困惑的特性，就是：`SQL` 语句的执行顺序跟其语句的语法顺序并不一致。`SQL` 语句的语法顺序是：

- `SELECT[DISTINCT]`
- `FROM`
- `WHERE`
- `GROUP BY`
- `HAVING`
- `UNION`
- `ORDER BY`

为了方便理解，上面并没有把所有的 `SQL` 语法结构都列出来，但是已经足以说明 `SQL` 语句的语法顺序和其执行顺序完全不一样，就以上述语句为例，其执行顺序为：

- `FROM`
- `WHERE`
- `GROUP BY`
- `HAVING`
- `SELECT`
- `DISTINCT`
- `UNION`
- `ORDER BY`

​    摘自：[十步完全理解 SQL](http://blog.jobbole.com/55086/)

<br />

# 查询语句 SELECT FROM 

## 语句解释
从表中选择数据
```SQL
SELECT
    attribute_name, ...
FROM
    table_name
```
## 查重语句
```SQL
SELECT
    attribute_name
FROM
    table_name
GROUP BY
    attribute_name
HAVING
    COUNT(attribute_name) > 1;
```

## 前N个语句
```SQL
SELECT
    *
FROM 
    table_name
ORDER BY 
    attribute_name DESC
LIMIT 
    n 
OFFSET 
    n
```

## CASE...END 判断语句
```SQL
CASE 
    WHEN 条件1 THEN 结果1
    WHEN 条件2 THEN 结果2
    WHEN 条件3 THEN 结果3
    .........
    WHEN 条件N THEN 结果N
END
```
<br />

# 筛选语句 WHERE 

## 语法
```SQL
SELECT 
    attribute_name
FROM
    table_name
WHERE 
    attribute_name operator value
```
属性名不要求相同。
## 运算符/通配符/操作符
| operator | 描述         |
| :------- | :----------- |
| =        | 等于         |
| <>       | 不等于       |
| >        | 大于         |
| <        | 小于         |
| >=       | 大于等于     |
| <=       | 小于等于     |
| BETWEEN  | 在某个范围内 |
| LIKE     | 搜索某种模式 |
<br />

# 分组语句 GROUP BY

## 语句解释
`GROUP BY`语句根据一个或多个列对结果集进行分组。

```SQL
SELECT 
    attribute_name
FROM
    table_name
WHERE 
    attribute_name operator value
GROUP BY 
    attribute_name
```
属性名不要求相同。

<br />

## HAVING子句
在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。
示例：
```SQL
SELECT 
    attribute_name, aggregate_function(attribute_name)
FROM 
    table_name
WHERE 
    attribute_name operator value
GROUP BY 
    attribute_name
HAVING 
    aggregate_function(attribute_name) operator value
```
<br />

# 排序语句 ORDER BY

`ORDER BY`语句用于根据指定的列对结果集进行排序。默认按照升序，添加`DESC`关键字可改成降序。

## 正序、逆序
`SQL`默认按照升序（`ASC`），添加`DESC`关键字可改成降序。



<br />

# 简单实操
## 查找重复的电子邮箱（难度：简单）
1. 创建表，表名 `accounts`，再添加 `records`。
    ```SQL
    CREATE TABLE email (
	ID INT NOT NULL PRIMARY KEY,
	Email VARCHAR(255)
	);
	
	INSERT INTO email VALUES('1','a@b.com');
	INSERT INTO email VALUES('2','c@d.com');
	INSERT INTO email VALUES('3','a@b.com');
	```
	Yeah，这就成了。
	![在这里插入图片描述](https://img-blog.csdnimg.cn/2019040219403986.png)
2. 查重
	```SQL
	SELECT 
	    Email 
	FROM 
	    accounts
	GROUP BY
	    Email
	HAVING
	    COUNT(Email)>1;
	```
	查找结果：
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190402194001508.png)

<br />

## 查找大国

1. 建表，表名 `world`，再添加 `records` 。
	```SQL
	CREATE TABLE World (
	name VARCHAR(50) NOT NULL,
	continent VARCHAR(50) NOT NULL,
	area INT NOT NULL,
	population INT NOT NULL,
	gdp INT NOT NULL
	);
	
	INSERT INTO World
	  VALUES('Afghanistan','Asia',652230,25500100,20343000);
	INSERT INTO World 
	  VALUES('Albania','Europe',28748,2831741,12960000);
	INSERT INTO World 
	  VALUES('Algeria','Africa',2381741,37100000,188681000);
	INSERT INTO World
	  VALUES('Andorra','Europe',468,78115,3712000);
	INSERT INTO World
	  VALUES('Angola','Africa',1246700,20609294,100990000);
	```
2. 查找大国（条件：国家的面积超过300万平方公里，或者(人口超过2500万并且 `gdp` 超过2000万)）
	```SQL
	SELECT
	    name, population, area
	FROM
	    world
	WHERE
	    area > 3000000 OR (population > 25000000 and GDP > 20000000);
	```
	Here we go!
	![在这里插入图片描述](https://img-blog.csdnimg.cn/20190402194829347.png)

<br />

# 系列

1. {% post_link installation-and-simple-instructions-of-sql 「SQL」1. 安装 MySQL %}
2. {% post_link basic-instructions-of-sql 「SQL」2. SQL格式规范、基本命令与范例 %} 
3. {% post_link data-types-of-MySQL 「SQL」3. MySQL 的数据类型  %}



<br />

**以上！**

<br />

