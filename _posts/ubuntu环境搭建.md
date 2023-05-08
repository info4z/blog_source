---
title: ubuntu环境搭建
date: 2023-02-17
excerpt: 记录一次 ubuntu 操作系统使用前的初始化过程
categories: 服务器
tags: ubuntu
---



## 一 : 远程连接

### (一) 密钥对

生成密钥对

```sh
$ ssh-keygen -t rsa -b 2048
```

将公钥文件上传至服务器端

```sh
$ ssh-copy-id [-p port] [user@]hostname
```

客户端尝试登录服务器

```sh
$ ssh [-p port] [user@]hostname
```

### (二) 设置新用户

创建用户

```shell
$ useradd -m -s /bin/bash zhang
```

设置密码

```shell
$ passwd zhang
```

删除用户

```shell
$ userdel -r zhang
```

sudo 授权

```shell
$ visudo
zhang ALL=(ALL) NOPASSWD:ALL
```



### (三) 安全配置

修改配置文件

```sh
$ vim /etc/ssh/sshd_config
```

禁止密码登录

```properties
#PasswordAuthentication yes
PasswordAuthentication no
```

禁止root远程登录

```properties
#PermitRootLogin yes
PermitRootLogin no
```

修改默认端口

```properties
#Port 22
Port 59527
```

限制 ssh 监听 IP 

```properties
#ListenAddress 0.0.0.0
ListenAddress 192.168.88.100	
# 只能通过192.168.88.100连接这台服务器
```



## 二 : 常用工具

### (一) 管理工具

lrzsz : 上传下载

```shell
$ apt install -y lrzsz
```

unzip : 解压缩

```shell
$ apt install -y unzip
```

rsync : 文件同步

```shell
$ apt install -y rsync
```

expect : 交互式脚本

```shell
$ apt install -y expect
```

### (二) 开发工具

openjdk

```sh
$ apt install -y openjdk-8-jdk
```

git

```shell
$ apt install -y git
```

maven : 项目管理

```shell
$ apt install -y maven
```



## 三 : 中间件

### (一) Nginx

```shell
$ apt install -y nginx
```

### (二) Redis

```shell
# 安装redis
$ apt install -y redis-server

# 修改配置文件
vim /etc/redis/redis.conf
bind 127.0.0.1 				# 如果需要远程连接,注释掉这行
daemonize no				# 如果需要后台运行,改成yes
requirepass 123456 			# 如果需要密码认证,放开这行的注释,这里密码设为123456
```

### (三) RabbitMQ

```shell
# 安装rabbit
$ apt install -y rabbitmq-server
```



## 四 : 数据库

### (一) MySQL

```sh
# 安装
$ apt install -y mysql-server

# 修改密码
$ alter user 'root'@'localhost' identified with caching_sha2_password by 'root';

# 新建账号,如果需要远程连接要用mysql_native_password,不然navicat会报错
$ create user 'zhang'@'%' identified with mysql_native_password by 'Zhang@123';

# 授权
$ grant all on *.* to 'zhang'@'%';
```



## 五 : 其他

### (一) phpmyadmin

```sh
$ apt install -y phpmyadmin
```



### (二) certbot

```sh
# 安装
$ apt install -y certbot
# 申请证书
$ certbot certonly
```

