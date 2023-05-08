---
title: Let's Encrypt
date: 2023-02-10
excerpt: 使用 Let's Encrypt + nginx 将网站升级到 HTTPS
categories: 服务器
tags: ssl
---





## 一 : 概述

`certbot` 是Let's Encrypt官方推荐的获取证书的客户端，可以帮我们获取免费的Let's Encrypt 证书。

`certbot` 支持所有  Unix 内核的操作系统。

## 二 : 安装使用

使用 yum 安装即可

```shell
[root@localhost]# yum list installed certbot
```

## 三 : 申请证书

申请证书之前要确保 443 端口没有被占用, 记得关掉 nginx

```shell
[root@localhost]# certbot certonly
# 1.standalone按1回车,webroot按2回车
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 1
# 2.输入联系邮箱
Enter email address (used for urgent renewal and security notices)
 (Enter 'c' to cancel): xxx@xxx.com
# 3.读一下声明,是否同意
(Y)es/(N)o: Y
# 4.是否分享
(Y)es/(N)o: Y
# 5.输入域名
Please enter the domain name(s) you would like on your certificate (comma and/or
space separated) (Enter 'c' to cancel): www.info4z.com
```

完毕后可以看到

```shell
Successfully received certificate.
Certificate is saved at: /etc/letsencrypt/live/www.info4z.com/fullchain.pem
Key is saved at:         /etc/letsencrypt/live/www.info4z.com/privkey.pem
```

## 四 : 编辑nignx配置文件

编辑对应的 nginx 配置文件

```shell
[root@localhost]# vim /etc/nginx/conf.d/www.info4z.com.conf
```

```nginx
# 替换新的证书
ssl_certificate      /etc/letsencrypt/live/www.info4z.com/fullchain.pem;
ssl_certificate_key  /etc/letsencrypt/live/www.info4z.com/privkey.pem;
```

修改完毕后重启 nginx 即可

## 五 : 证书续签

Let's Encrypt 提供的证书只有90天的有效期，我们必须在证书到期之前，重新获取这些证书

certbot 给我们提供了一个很方便的命令

```shell
certbot renew
```

值得注意的是, 我们生成证书的时候使用的是 `standalone` 模式, 验证域名的时候, 需要启用443端口, 否则会报端口占用的错误

证书是90天才过期, 在过期之前执行更新操作就可以了; 这件事情可以交给定时任务完成, linux 系统上有 cron 可以来搞定这件事情

```shell
# 新建 certbot-auto-renew-cron
vim certbot-auto-renew-cron
# --pre-hook: 表示执行更新操作之前要做的事情
# --post-hook: 表示执行更新操作完成后要做的事情
# cron表达式(分 时 日 月 周几): 从1月开始每个2个月的1日2点15分执行一次
15 2 1 1/2 ? certbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
```

```shell
# 启动定时任务
crontab certbot-auto-renew-cron
```





