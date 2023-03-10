---
title: mysql数据统计
excerpt: 统计数据占用磁盘空间的大小
date: 2023-01-06
categories: 数据库
tags: mysql
---





## 一 : 库表结构

* MySQL 的 information_schema 数据库, 保存着数据库的容量和使用信息, 可查询数据库中每个表占用的空间, 表记录的行数

* TABLES 表结构

  ```sql
  CREATE TEMPORARY TABLE `TABLES` (
    `TABLE_CATALOG` varchar(512) NOT NULL DEFAULT '',
    `TABLE_SCHEMA` varchar(64) NOT NULL DEFAULT '',
    `TABLE_NAME` varchar(64) NOT NULL DEFAULT '',
    `TABLE_TYPE` varchar(64) NOT NULL DEFAULT '',
    `ENGINE` varchar(64) DEFAULT NULL,
    `VERSION` bigint(21) unsigned DEFAULT NULL,
    `ROW_FORMAT` varchar(10) DEFAULT NULL,
    `TABLE_ROWS` bigint(21) unsigned DEFAULT NULL,
    `AVG_ROW_LENGTH` bigint(21) unsigned DEFAULT NULL,
    `DATA_LENGTH` bigint(21) unsigned DEFAULT NULL,
    `MAX_DATA_LENGTH` bigint(21) unsigned DEFAULT NULL,
    `INDEX_LENGTH` bigint(21) unsigned DEFAULT NULL,
    `DATA_FREE` bigint(21) unsigned DEFAULT NULL,
    `AUTO_INCREMENT` bigint(21) unsigned DEFAULT NULL,
    `CREATE_TIME` datetime DEFAULT NULL,
    `UPDATE_TIME` datetime DEFAULT NULL,
    `CHECK_TIME` datetime DEFAULT NULL,
    `TABLE_COLLATION` varchar(32) DEFAULT NULL,
    `CHECKSUM` bigint(21) unsigned DEFAULT NULL,
    `CREATE_OPTIONS` varchar(255) DEFAULT NULL,
    `TABLE_COMMENT` varchar(2048) NOT NULL DEFAULT ''
  ) ENGINE=MEMORY DEFAULT CHARSET=utf8
  ```

* 重点字段说明

  | 字段         | 解释            |
  | ------------ | --------------- |
  | TABLE_SCHEMA | 数据库名        |
  | TABLE_NAME   | 表名            |
  | ENGINE       | 存储引擎        |
  | TABLE_ROWS   | 记录数          |
  | DATA_LENGTH  | 数据大小(单位B) |
  | INDEX_LENGTH | 索引大小        |



## 二 : 查询示例

### (一) 数据总占用量

* 求和 => 单位转换 => 加单位

  ```sql
  select 
  	concat(round(sum(DATA_LENGTH/1024/1024/1024),2),'GB') as data 
  from information_schema.TABLES
  ```

### (二) 每个表占用量

* table_name, table_rows, data_length

  ```sql
  select 
  	table_name, 
  	concat(round(sum(data_length/1024/1024/1024),2),'G') as data 
  from information_schema.tables 
  where table_schema='dbname' 
  group by table_name 
  order by data desc;
  ```

  