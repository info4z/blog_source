---
title: mysql8.0修改密码
date: 2023-03-03
excerpt: 与5.7相比有些许不同,这里简单记录一下过程
categories: 数据库
tags: mysql
---



## 一 : 置空密码

编辑 `/etc/my.cnf`

```properties
# 插入一行, 跳过授权表
skip-grant-tables
```

重启后登录

```sh
# 重启mysql
$ systemctl restart mysqld
# 登录
$ mysql -uroot -p
```

置空密码

```sql
update mysql.user set authentication_string='' where user='root';
```



## 二 : 设置密码

取消跳过授权

```properties
# 其实也没啥必要删,注释掉即可
# skip-grant-tables
```

重启后登录

```shell
# 重启
$ systemctl restart mysqld
# 登录
$ mysql -uroot -p
```

重置密码

```sql
-- 重置密码, mysql8.0开始需要符合密码安全规范
alter user 'root'@'localhost' identified by 'Qwer@123';

-- 如果需要设置简单的,修改一下密码校验规则即可
show variables like 'validate_password%';
set global validate_password.check_user_name = OFF;
set global validate_password.length = 4;
set global validate_password.policy = 0;
-- 然后再设置就成功了
alter user 'root'@'localhost' identified by 'root';
```

创建普通账户

```sql
-- 指定加密规则,不然navicat会报错
create user 'zhang'@'%' identified with mysql_native_password by 'Zhang@123';
-- 最好检查一下加密规则是否符合预期
select Host,user,authentication_string,password_expired,plugin from mysql.user;
```

删除用户

```sql
drop user 'zhang'@'%';
```

