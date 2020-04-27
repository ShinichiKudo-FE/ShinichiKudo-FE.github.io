---
title: Node学习(十) --数据库入门
date: 2019-04-21 21:43:12
tags: "Node"
categories: "Node"
---

## 数据库的分类

1.**文件数据库** 它的优点是：数据保存在单一文件中，因此部署十分方便，常用来嵌入到APP中保存数据，以及手机中的短信、通话记录等。

常用的有SQLite、Access等。

2.**关系型数据库** 关系型数据库的功能强大，适用场景丰富，它的特点是数据之间互相是关联的。

比如淘宝，一个用户下有个人信息、订单信息、聊天记录等，可以分别存储在不同表中，通过用户ID相互关联起来。但它的性能不是最强的。

常用有SQL。

3.**分布式数据库** 它的特点是可以安装在多台机器上，可以将查询等操作分散开，合理利用服务器资源。

还可以将数据设置多个备份，一旦其中一台服务器异常，可以由其他服务器同步数据。

常用的有MongoDB。

4.**NoSQL** 它只能支持简单查询，但性能强于关系型数据库，常用来做接口、数据缓存等。

常用的有MongoDB、Memcached、Redis等。

## SQL语句

结构化查询语言(Structured Query Language)简称SQL，它主要用来存取、查询、更新和管理关系数据库系统。

SQL 分为两个部分：数据定义语言 (DDL)和数据操作语言 (DML) 。

## DDL语句

CREATE DATABASE - 创建新数据库
ALTER DATABASE - 修改数据库
CREATE TABLE - 创建新表
ALTER TABLE - 变更（改变）数据库表
DROP TABLE - 删除表
CREATE INDEX - 创建索引（搜索键）
DROP INDEX - 删除索引

## DDM语句

数据操作语言 (DML) 是最常用的操作，即增删改查，它包括如下部分：

1.增INSERT INTO - 向数据库表中插入数据
INSERT INTO <表> (字段, ...) VALUES(值, ...);
INSERT INTO user_table (username, password) VALUES('lee', '123456');
在user_table表中插入一条数据。

2.删DELETE - 从数据库表中删除数据
DELETE FROM <表> WHERE 条件;
DELETE FROM user_table WHERE ID=2;
删除user_table表中ID为2的项目。

3.改UPDATE - 更新数据库表中的数据
UPDATE <表> SET 字段=新值,字段=新值,... WHERE 条件;
UPDATE user_table SET password='888888', username='chen' WHERE ID=1;
将user_table表中ID为1的用户名和密码更新。

4.查SELECT - 从数据库表中获取数据
SELECT 字段列表 FROM <表> WHERE 条件 ORDER BY 字段 LIMIT 30,30;
SELECT ID FROM user_table WHERE username=‘lee’ ORDER BY ID ASC LIMIT 0,10;
查询user_table表中username为lee的用户的ID，取0~10个，按升序排列。

## 数据库索引

数据库的索引需要作用是提高查询性能，另外还可以对数据进行限制，如限制为“唯一”。

索引的类型有四种，如下：

1.主键

具有“唯一”和“索引”的双重优点。

2.唯一

定义字段为唯一，即不可重复。

3.索引

优点：提高查询性能，相当于书的目录。

缺点：降低增、删、改的性能，因为这些操作可能会触发数据库整理、重建索引。另外，索引也会占用磁盘空间。

4.全文索引

适合文本搜索，常用在搜索引擎中。
