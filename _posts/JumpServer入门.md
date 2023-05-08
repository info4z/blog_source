---
title: JumpServer入门
date: 2023-03-10
excerpt: 发现一个开源堡垒机, 简单体验一下
categories: 服务器
tags: 跳板机
---



## 一 : 概述

JumpServer 是广受欢迎的开源堡垒机，是符合 4A 规范的专业运维安全审计系统。

JumpServer 的产品特色包括：

- 开源：零门槛，线上快速获取和安装；
- 分布式：轻松支持大规模并发访问；
- 无插件：仅需浏览器，极致的 Web Terminal 使用体验；
- 多云支持：一套系统，同时管理不同云上面的资产；
- 云端存储：审计录像云端存储，永不丢失；
- 多租户：一套系统，多个子公司和部门同时使用；
- 多应用支持：数据库，Windows 远程应用，Kubernetes。



## 二 : 安装

在线安装

```shell
# 进入/opt
[root@localhost ~]# cd /opt/

# 下载安装
[root@localhost opt]# curl -sSL https://resource.fit2cloud.com/jumpserver/jumpserver/releases/latest/download/quick_start.sh | bash
>>> The Installation is Complete
For more commands, you can enter jmsctl --help to view help information.

# 同时可以看到以下文件
[root@localhost opt]# ll
total 0
drwx--x--x. 4 root root  28 Mar 20 15:22 containerd
drwx------. 3 root root  20 Mar 20 15:22 jumpserver
drwxr-xr-x. 8 root root 267 Mar 20 15:22 jumpserver-installer-v3.1.0
```

控制命令

```shell
cd /opt/jumpserver-installer-v3.1.0
# 启动
./jmsctl.sh start
# 停止
./jmsctl.sh down
# 卸载
./jmsctl.sh uninstall
# 帮助
./jmsctl.sh -h
```

配置文件路径为 : `/opt/jumpserver/config/config.txt`

```shell
# mysql
DB_HOST=mysql
DB_PORT=3306
DB_USER=root
DB_PASSWORD=MzA1MTRkNTYtYWY3Mi1kMTQ1LT
DB_NAME=jumpserver
# redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=MzA1MTRkNTYtYWY3Mi1kMTQ1LT
```

环境访问

```yaml
地址: http://<JumpServer服务器IP地址>:<服务运行端口>
用户名: admin
密码: admin
```

## 三 : 使用

### (一) 用户管理

创建用户 : 添加 dev(开发), star(运维), test(测试); 系统角色 => 用户

创建用户组 : 这个就不多说了, 很简单

### (二) 资产管理

1. 资产列表
2. 在资产树下面创建节点
3. 点击创建 
4. 选择主机 : Linux 
5. 基本内容 : 名称命名规范 `${IP}-${服务器名称}` 
6. 账号 : 添加用于认证的账号密码(尽量不要使用特权账号)

### (三) 权限管理

1. 资产授权
2. 创建
3. 基本 : 授权名称(例如: 运维人员授权)
4. 用户 : 可以选择某个用户或者是某个用户组
5. 资产 : 可以选择某个资产或者某个节点
6. 账号 : 尽量不要选择同名账号, 避免新手混淆
7. 权限 : 按需求授权即可