---
title: mysql8.0日志
excerpt: 主要涉及到错误日志,查询日志,慢查询日志和二进制日志
date: 2023-03-17
categories: 数据库
tags: mysql
---



## 一 : 错误日志

错误日志(log_error)是**默认开启**的, 当数据库出现任何故障导致无法正常使用时, 建议首先查看此日志 

```sql
mysql> show variables like '%log_error%';
+----------------------------+----------------------------------------+
| Variable_name              | Value                                  |
+----------------------------+----------------------------------------+
| binlog_error_action        | ABORT_SERVER                           |
| log_error                  | /var/log/mysql/error.log               |
| log_error_services         | log_filter_internal; log_sink_internal |
| log_error_suppression_list |                                        |
| log_error_verbosity        | 2                                      |
+----------------------------+----------------------------------------+
```

该日志主要包含的信息有

* 服务器启动和关闭过程中的信息
* 服务器运行过程中的错误信息
* 时间调度器运行一个时间时产生的信息
* 在从服务器上启动从服务器进程时产生的信息



## 二 : 普通日志

普通日志(general_log)主要记录了用户的 DDL(create, drop, alter),  DQL(select) 和 DML(insert, update, delete) 语句, **默认未开启**

```sql
mysql> show variables like '%general_log%';
+------------------+--------------------------------------------+
| Variable_name    | Value                                      |
+------------------+--------------------------------------------+
| general_log      | OFF                                        |
| general_log_file | /var/lib/mysql/iv-ycco47wj7w7grb1bc17k.log |
+------------------+--------------------------------------------+

-- 输出方式: file(文件),table(表)
mysql> show variables like '%log_output%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_output    | FILE  |
+---------------+-------+
```

### (一) 临时开启

设置全局变量

```sql
-- 日志记录功能: on(开启),off(关闭)
mysql> set global general_log=on;
-- 文件位置
mysql> set global general_log_file='/var/log/mysql/general.log';
```

### (二) 永久开启

编辑配置文件

```shell
$ sudo vim /etc/my.cnf
# 开启查询日志: 1(开启), 0(关闭)
general_log=1
# 设置日志文件名, 默认文件名问hostname.log
general_log_file=/var/log/mysql/general.log
```

重启mysql服务

```shell
$ sudo systemctl restart mysqld
```



## 三 : 慢查询日志

慢查询日志(slow_query_log)记录所有执行时间超过参数 long_query_time 设置值并且扫描记录数不小于 min_examined_row_limit 的所有的 SQL 语句的日志, **默认未开启**

```sql
mysql> show variables like '%slow_query_log%';
+---------------------+----------------------------+
| Variable_name       | Value                      |
+---------------------+----------------------------+
| slow_query_log      | OFF                         |
| slow_query_log_file | /var/lib/mysql/vm-slow.log |
+---------------------+----------------------------+
```

long_query_time 默认值为 10s, 最小值为 0

```sql
mysql> show variables like '%long_query_time%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

### (一) 临时开启

设置全局变量

```sql
mysql> set global slow_query_log=on;
mysql> set global slow_query_log_file=/var/log/mysql/slow.log;
mysql> set global long_query_time=2;
```

### (二) 永久开启

开启慢查询日志

```shell
vim /etc/my.cnf
# 开启慢查询日志
slow_query_log=1
# 查询时长参数
long_query_time=2
```

重启mysql

```shell
$ sudo systemctl restart mysql
```



## 四 : 二进制日志

二进制日志(log_bin)记录了所有 DDL 语句和 DML, 但是不包括数据查询(select, show), 默认**开启**; 可用于灾难时的数据恢复和主从复制

查看默认值

```sql
mysql> show variables like '%log_bin%';
+---------------------------------+-----------------------------+
| Variable_name                   | Value                       |
+---------------------------------+-----------------------------+
| log_bin                         | ON                          | 
| log_bin_basename                | /var/lib/mysql/binlog       |	-- 日志文件
| log_bin_index                   | /var/lib/mysql/binlog.index |	-- 日志索引
| log_bin_trust_function_creators | OFF                         |
| log_bin_use_v1_row_events       | OFF                         |
| sql_log_bin                     | ON                          |
+---------------------------------+-----------------------------+
```

日志格式

```sql
mysql> show variables like '%binlog_format%';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| binlog_format | ROW   |
+---------------+-------+
```

### (一) 日志查看

二进制文件不能直接读取, 需要通过 `mysqlbinlog` 命令查看

```shell
mysqlbinlog [option] filename
option:
	-d: 指定数据库名称, 只列出指定数据库相关操作
	-o: 忽略掉日志中的前n行指令
	-v: 将事件(数据变更)重构为SQL
	-w: 将事件(数据变更)重构为SQL语句并输出注释
```

### (二) 日志还原

命令操作

```shell
mysqlbinlog [option] filename | mysql -uroot -proot
option:
	--start-datetime: 恢复数据的起始时间
	--stop-datatime: 恢复数据的结束时间
	--start-position: 恢复数据的开始位置
	--stop-position: 恢复数据的结束位置
```

