---
title: 【MySQL】MySQL 安装及 SQL 入坑
date: 2019-05-02 20:59:22
tags:
	- SQL
	- MySQL
	- setup
categories:	
	- SQL
---

**简述**：四月份开始学习`SQL`，一个新坑的开始，填不完的坑。直到现在才整理了一篇博文，因为坑太多了。。。

<!-- more -->
<br />

# 软件安装及初始化

## 安装
下载地址：[here](https://dev.mysql.com/downloads/mysql/)
步骤：
1. 对压缩包解压缩，并放到某个本地位置，如`D:\Program Files\MySQL`
2. 进入`D:\Program Files\MySQL`（这里修改为自己的安装目录），新建文件`my.ini`。打开并输入再保存：
    ```
    [mysql]
    # 设置mysql客户端默认字符集
    default-character-set=utf8
 
    [mysqld]
    # 设置3306端口
    port = 3306
    # 设置mysql的安装目录
   basedir=D:\\Program Files\\MySQL
   # 允许最大连接数
   max_connections=20
   # 服务端使用的字符集默认为8比特编码的latin1字符集
   character-set-server=utf8mb4
   # 创建新表时将使用的默认存储引擎
   default-storage-engine=INNODB
   ```

3. **以管理员打开命令行窗口（CMD），很重要。**
4. 切换路径至`D:\Program Files\MySQL\bin`
5. 初始化数据库：`mysqld --defaults-file=D:\Program Files\MySQL\my.ini --initialize --console`。执行完成后，会输出` root `用户的初始默认密码，如：
    ```sql
    2018-04-20T02:35:05.464644Z 5 [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: BAsCY5ws,hjt
   ```
   `BAsCY5ws,hjt`就是初始密码，后续登录需要用到。
6. 启动：`net start mysql`。
7. 停止：`net stop mysql`。

## 简单命令
命令行窗口目录： `D:\Program Files\MySQL\bin`
1. 登录：`mysql -u root -p`。输入之前记下的 temporary password，回车之后就登录了。
2. **修改密码（超难）**：`set password for 'root'@'localhost' = 'xxxxxxx;'`。**不要忘记`;` 结尾**。
    网上TMD教程全是错的（`set password for 'root'@'localhost' = password('xxxxxxx')`）。
3. 显示已有的数据库：mysql>`show databases;`。
    ```sql
    mysql> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    +--------------------+
    4 rows in set (0.08 sec)
    ```
4. 新建数据库：mysql>`create database test;`。
5. 使用数据库：mysql>`use test;`。

<br />

-- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- **之后就基本是废话了** -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --

# 数据库基础知识

## 数据库定义
数据库（Database）是按照数据结构来组织、存储和管理数据的仓库。
每个数据库都有一个或多个不同的 API 用于创建，访问，管理，搜索和复制所保存的数据。

## 关系型数据库
所谓的关系型数据库，是建立在关系模型基础上的数据库，借助于集合代数等数学概念和方法来处理数据库中的数据。关系数据库管理系统(Relational Database Management System)的特点：

1. 数据以表格的形式出现
2. 每行为各种记录名称
3. 每列为记录名称所对应的数据域
4. 许多的行和列组成一张表单
5. 若干的表单组成database

## 行
>行：一行（=元组，或记录）是一组相关的数据，例如某个用户的年龄，出生日期，手机号码等数据。`

## 列
>列: 一列(数据元素) 包含了相同的数据, 例如学号。`

## 主键
>主键：**主键唯一**。一个数据表中只能包含一个主键。你可以使用主键来查询数据。

## 外键
>外键：用于关联两个表。

<br />
# MySQL数据库管理系统
## 数据库
>数据库: 数据库是一些关联表的集合。
## 数据表
>数据表: 表是数据的矩阵。在一个数据库中的表看起来像一个简单的电子表格。
## 视图
>虚拟的表。本身不存储数据，而是按指定的方式进行查询。
## 存储过程
>为以后的使用而保存的一条或多条MySQL语句的集合。



<br />

以上！