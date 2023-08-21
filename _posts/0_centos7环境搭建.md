---
title: centos7环境搭建
excerpt: 记录一次centos7操作系统使用前的初始化过程
date: 2023-03-31
categories: 服务器
tags: [操作系统, centos7]
---



## 一 : 远程连接

### (一) 密钥对

生成密钥对

```sh
ssh-keygen -t rsa -b 2048
```

将公钥文件上传至服务器端

```sh
ssh-copy-id [-p port] [user@]hostname
```

客户端尝试登录服务器

```sh
ssh [-p port] [user@]hostname
```

### (二) 设置新用户

创建用户

```shell
useradd -m -s /bin/bash zhang
```

设置密码

```shell
passwd zhang
```

sudo 授权

```shell
visudo
zhang ALL=(ALL) NOPASSWD: ALL
```

删除用户

```shell
userdel -r zhang
```

### (三) 安全配置

修改配置文件

```sh
[root@local ~]# vim /etc/ssh/sshd_config
```

禁止root远程登录

```properties
#PermitRootLogin yes
PermitRootLogin no
```

禁止密码登录

```properties
#PasswordAuthentication yes
PasswordAuthentication no
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

重启 sshd 生效

```shell
systemctl restart sshd
```

### (四) 修改主机名



## 二 : 常用工具

### (一) 管理工具

lrzsz : 上传下载

```shell
yum install -y lrzsz
```

unzip : 解压缩

```shell
yum install -y unzip
```

rsync : 文件同步

```shell
yum install -y rsync
```

expect : 交互式脚本

```shell
yum install -y expect
```

### (二) 开发工具

openjdk

```sh
yum install -y java-1.8.0-openjdk-devel.x86_64
yum install -y java-11-openjdk-devel.x86_64
```

git

```shell
wget https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.41.0.tar.gz
tar -xf 
yum -y install gcc openssl openssl-devel curl curl-devel unzip perl perl-devel expat expat-devel zlib zlib-devel asciidoc xmlto gettext-devel openssh-clients
./configure --prefix=/usr/local/git
make
make install

vim /etc/profile
export PATH=/usr/local/git/bin:$PATH
source /etc/profile

# 记住密码
git config credential.helper store
```

maven : 项目管理

```shell
# 下载
wget https://dlcdn.apache.org/maven/maven-3/3.9.1/binaries/apache-maven-3.9.1-bin.tar.gz
# 解压
tar -xf apache-maven-3.9.1-bin.tar.gz
# 配置环境变量
vim /etc/profile
# 导出maven
export MAVEN_HOME=/usr/local/maven/apache-maven-3.9.1
export PATH=$PATH:$MAVEN_HOME/bin
# 重新加载
source /etc/profile

# 修改配置文件
vim /usr/local/maven/apache-maven-3.9.1/conf/settings.xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```



## 三 : 数据库

### (一) MySQL8.0

```sh
# 下载
wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.32-1.el7.x86_64.rpm-bundle.tar
# 卸载mariadb
rpm -qa | grep mariadb
yum remove -y mariadb-libs-5.5.68-1.el7.x86_64
# 安装
rpm -ivh mysql-community-server-8.0.32-1.el7.x86_64.rpm 
# 可能会缺个依赖, 用 yum 补一下就好
yum install -y libaio

# 修改配置文件(sql_mode和最大连接数)
vim /etc/my.cnf
sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
max_connections=2000

# 修改密码
vim /etc/my.cnf
skip-grant-tables
# 重启mysql
systemctl restart mysqld
# 置空密码
mysql
> update mysql.user set authentication_string='' where user='root';
# 取消skip-grant-tables后重启
systemctl restart mysqld
# 充值密码
alter user 'root'@'localhost' identified by 'Qwer@123';
# 创建用户
create user 'zhang'@'%' identified with mysql_native_password by 'Zhang@123';
# 授权
grant all on *.* to 'zhang'@'%';
# 刷新权限
flush privileges;
```

### (二) phpmyadmin

```shell
wget https://files.phpmyadmin.net/phpMyAdmin/5.2.1/phpMyAdmin-5.2.1-all-languages.zip
```



## 四 : 中间件

### (一) Nginx

```shell
yum install -y nginx
```

### (二) Redis

```shell
# 安装redis
yum install -y redis.x86_64

# 修改配置文件
vim /etc/redis.conf
bind 127.0.0.1 				# 如果需要远程连接,注释掉这行
daemonize no				# 如果需要后台运行,改成yes
requirepass 123456 			# 如果需要密码认证,放开这行的注释,这里密码设为123456
```

### (三) RabbitMQ

```shell
# 下载MQ
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.28/rabbitmq-server-3.7.28-1.el7.noarch.rpm
# 下载erlang
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v21.3.8.21/erlang-21.3.8.21-1.el7.x86_64.rpm
# 安装erlang
rpm -ivh erlang-21.3.8.21-1.el7.x86_64.rpm
# 安装socat
yum install -y socat
# 安装MQ
rpm -ivh rabbitmq-server-3.7.28-1.el7.noarch.rpm

# 开启web插件
rabbitmq-plugins enable rabbitmq_management
# 访问 http://139.224.105.67:15672/
# 账号密码: guest/guest

# 下载延迟插件
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez
# 查找插件位置
rpm -ql rabbitmq-server-3.7.28-1.el7.noarch | grep plugins
# 安装延迟插件
cp ./rabbitmq_delayed_message_exchange-3.8.0.ez /usr/lib/rabbitmq/lib/rabbitmq_server-3.7.28/plugins/
# 激活延迟插件
rabbitmq-plugins list
rabbitmq-plugins enable rabbitmq_delayed_message_exchange

# nginx配置
location /rabbitmq/ {
    port_in_redirect on;
    proxy_redirect off;
    proxy_pass http://127.0.0.1:15672/;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header User-Agent $http_user_agent;
    proxy_set_header X-Forwarded-Proto $scheme;
}

location /rabbitmq/api/ {
    rewrite ^ $request_uri;
    rewrite ^/rabbitmq/api/(.*) /api/$1 break;
    return 400;
    proxy_pass http://127.0.0.1:15672$uri;
    proxy_buffering off;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

### (四) Nacos

下载安装

```shell
# 下载
wget https://github.com/alibaba/nacos/releases/download/2.2.2/nacos-server-2.2.2.tar.gz
# 解压
tar -xf nacos-server-2.2.2.tar.gz
```

编辑配置文件

```shell
vim conf/application.properties
```

```properties
# 数据库类型
spring.datasource.platform=mysql
# 数据库数量
db.num=1
# 数据库配置
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=zhang
db.password.0=Zhang@123

# 开启鉴权
nacos.core.auth.enabled=true
# 认证白名单配置，请求头携带 key:value 即可忽略鉴权，相当于后门，应用场景少，其中 key和value不能为空
nacos.core.auth.server.identity.key=nacos
nacos.core.auth.server.identity.value=nacos
### The default token (Base64 String): Base64 在线编码解码： https://base64.us/
nacos.core.auth.plugin.nacos.token.secret.key=VGhpc0lzTXlDdXN0b21TZWNyZXRLZXkwMTIzNDU2Nzg=
```

单机启动

```shell
bash bin/startup.sh -m standalone
# 访问 http://139.224.105.67:8848/nacos
```

集群启动

```shell
vim conf/cluster.conf
```

```
#example
192.168.16.101:8847
192.168.16.102
192.168.16.103
```

### (五) Powerjob

下载安装

```shell
# 下载源码
wget https://github.com/PowerJob/PowerJob/archive/refs/tags/v4.0.1.zip
# 解压
unzip v4.0.1.zip
```

创建数据库

```mysql
CREATE DATABASE IF NOT EXISTS `powerjob` DEFAULT CHARSET utf8mb4;
```

修改配置文件

```shell
# daily为默认配置文件,生产环境可以用product
# PowerJob/powerjob-server/powerjob-server-starter/src/main/resources/application-daily.properties
vim application-daily.properties
spring.datasource.core.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.core.jdbc-url=jdbc:mysql://localhost:3306/powerjob?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
spring.datasource.core.username=root
spring.datasource.core.password=root
```

编译打包

```shell
# 构建调度服务器: powerjob-server.jar 
mvn clean package -U -Pdev -DskipTests
```

启动程序

```shell
# 运行:生产话你就能够需要指定product
nohup java -jar xxx.jar --spring.profiles.active=product &
```

访问: http://127.0.0.1:7700/

### (六) XXL-JOB

```shell
# 下载源码
wget https://github.com/xuxueli/xxl-job/archive/refs/tags/2.4.0.tar.gz
# 解压
mv 2.4.0.tar.gz xxl-job-2.4.0.tar.gz
tar -xf xxl-job-2.4.0.tar.gz
# 修改xxl-job-admin配置文件
vim application.properties
# 打包
mvn clean package
# 运行
java -jar xxl-job-admin-2.4.0.jar
# 访问 http://139.224.105.67:8080/xxl-job-admin/
```

### (七) Elasticsearch

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.10-x86_64.rpm
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.10-x86_64.rpm.sha512
rpm -ivh elasticsearch-7.17.10-x86_64.rpm
```



## 五 : 未完待续

