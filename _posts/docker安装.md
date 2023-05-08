---
title: docker安装
excerpt: 记录一下docker的安装过程
date: 2021-01-15
categories: 容器化技术
tags: [docker, docker入门]
---



## 一 : docker 版本命名

docker 在 1.13 版本之后, 从2017年的3月1日开始, 版本命名规则变为如下

| 项目        | 说明         |
| ----------- | ------------ |
| 版本格式    | YY.MM        |
| Stable 版本 | 每个季度发行 |
| Edge版本    | 每个月发行   |

同时 docker 划分为 CE 和 EE

* CE : 即社区版, 免费, 支持周期三个月
* EE : 即企业版, 强调安全, 付费使用



## 二 : docker 安装

官方网站上有各种环境下的安装指南, 这里主要介绍 docker CE 在 linux 上的安装

官方安装指南地址: [https://docs.docker.com/engine/installation](https://docs.docker.com/engine/installation)

系统要求 : docker CE 支持 64 位版本 Centos 7, 并且要求内核版本不低于 3.10

### (一) 安装

旧版本的 Docker 称为 docker 或者 docker-engine, 使用以下命令卸载旧版本

```shell
$ sudo yum remove docker docker-common docker-selinux docker-engine
```

使用yum安装即可

```shell
$ sudo yum install docker-ce
```

注意 : 如果安装的是centos 7 minimal 版本, 执行安装提示没有可用软件包, 需要安装必要的软件依赖及更新增加 docker-ce yum 源

```shell
$ sudo wget https://download.docker.com/linux/centos/docker-ce.repo
```

在测试或开发环境中 docker 官方为了简化安装流程, 提供了一套便捷的安装脚本, CentOS 系统上可用使用这套脚本安装

```shell
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh --mirror Aliyun
# 执行这个命令后,脚本就会自动的将一切准备工作做好,并且把 docker CE 的 Edge 版本安装在系统中
```

### (二) 启动测试

启动 docker-ce

```shell
$ sudo systemctl start docker		# 启动docker服务
$ sudo systemctl enable docker		# 设置开机启动
```

默认情况下, docker 命令会使用 unix socket 与 docker 引擎通讯; 而只有 root 用户和 docker 组的用户才可用访问 docker 引擎的 unix socket; 一般 linux 系统上不会直接使用 root 用户进行操作; 因此, 需要将使用 docker 的用户加入 docker 用户组

```shell
$ cat /etc/group | grep docker		# 通常会自动创建docker组
$ sudo groupadd docker 				# 如果没有就自己建立
$ sudo usermod -aG docker $USER 	# 将当前用户加入docker组
$ id $USER							# 查看是否添加
```

测试 docker 是否安装正确, 若能正常输出信息, 则说明安装成功

```shell
$ docker run hello-world 			# 启动一个基于hello-world镜像的容器
$ newgrp docker						# 如果仍然报权限不足,切换有效组(好像再切回原来的组还照样可以使用)
```



## 三 : 卸载 docker

删除 docker 安装包

```shell
$ sudo yum remove docker-ce
```

删除 docker 镜像

```shell
$ sudo rm -rf /var/lib/docker
```



## 四 : 镜像加速器

国内从 docker hub 拉取镜像有时会遇到困难, 此时可用配置镜像加速器; 

docker 官方和国内很多云服务商都提供了国内加速器服务, 例如 : 
* docker 官方提供的中国 registry mirror : https://registry.docker-cn.com
* 阿里云加速器
* DaoCloud加速器 : http://f1361db2.m.daocloud.io
* 163加速器 : http://hub-mirror.c.163.com

### (一) 配置镜像加速器

对于使用 systemd 的系统, 请在 `/etc/docker/daemon.json` 中写入如下内容(如果文件不存在需要新建该文件)

```json
{
    "registry-mirrors":[
        "http://hub-mirror.c.163.com"
    ]
}
```

重新启动服务生效

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### (二) 检查加速器是否生效

配置加速器之后, 如果拉取镜像仍然十分缓慢, 请手动检查加速器配置是否生效, 在命令行执行 `docker info`, 如果从结果中看到了如下内容, 说明配置成功

```shell
$ docker info
Registry Mirrors:
  http://hub-mirror.c.163.com/
```

