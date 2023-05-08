---
title: docker数据挂载
excerpt: 数据卷和主机目录分别怎么挂载?有什么好处?
date: 2021-02-26
categories: 容器化技术
tags: [docker, docker进阶]
---



## 一 : 数据管理

在容器中管理数据主要有两种方式
* 数据卷 (Volumes)
* 挂载主机目录 (Bind mounts)



## 二 : 数据卷

### (一) 概述

**数据卷**是一个可供一个或多个容器使用的特殊目录, 它绕过 UFS(Unix文件系统), 可以提供很多有用的**特性**

1. 数据卷可以在容器之间共享和重用
2. 对数据卷的修改会立马生效
3. 对数据卷的更新, 不会影响镜像
4. 数据卷默认会一直存在, 即使容器被删除

Docker 中提供了**两种挂载方式**, `-v` 和 `--mount` , 这两种方式该如何选择呢 ?

* Docker 新手应该选择 `--mount` 参数
* 经验丰富的 Docker 使用者对 `-v` 或者 `--volume` 已经很熟了, 但是推荐使用 `--mount` 参数

**注意 :** 数据卷的使用, 类似于 Linux 下对目录或文件进行 mount, 镜像中的被指定为挂载点的目录中的文件会隐藏掉, 能显示看的是挂载的数据卷

### (二) 创建数据卷

创建一个数据卷

```shell
$ docker volume create my-volume
```

查看所有的数据卷

```shell
$ docker volume ls
```

查看指定数据卷的信息

```shell
$ docker volume inspect my-volume
```

删除数据卷

```shell
$ docker volume rm my-volume
```

### (三) 挂载数据卷

启动一个挂载数据卷的容器, 在用 `docker run` 命令的时候, 使用 `--mount` 将**数据卷**挂载到容器里; 在一次 `docker run` 中可以挂载**多个**数据卷

创建一个名为 session-web 的容器, 并加载一个数据卷到容器的 `/webapp` 目录

```shell
$ docker run --name session-web -d -p 8888:8080 \
# -v my-volume:/webapp \
--mount source=my-volume,target=/webapp \
session-web:latest
```

数据卷是被设计用来持久化数据的, 它的生命周期独立于容器, Docker 不会在容器被删除后自动删除数据卷, 并且也不存在垃圾回收这样的机制来处理没有任何引用的数据卷; 如果需要在删除容器的同时移除数据卷; 可以在删除容器的时候使用 `docker rm -v` 这个命令

无主的数据卷可能会占据很多空间, 要清理可以使用以下命令

```shell
$ docker volume prune
```



## 三 : 挂载主机目录

### (一) 本地到容器

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去

```shell
$ docker run --name session-web -d -p 8888:8080 \
# -v /src/webapp:/opt/webapp \
--mount type=bind,source=/src/webapp,target=/opt/webapp \
session-web.latest
```

示例命令加载主机的 `/src/webapp` 目录到容器的 `/opt/webapp` 目录; 这个功能在进行测试的时候十分方便, 比如用户可以放置一些程序到本地目录中, 来查看容器是否正常工作

注意 :

* 本地目录的路径必须是绝对路径
* 以前 : 使用 `-v` 参数时如果本地目录不存在 Docker 会自动为你创建一个文件夹
* 现在 : 使用 `--mount` 参数时如果本地目录不存在, Docker 会报错
* **Docker 挂载主机目录的默认权限是读写, 用户也可以通过增加 readonly 指定为只读**

### (二) 容器到本地

`--mount` 标记也可以从主机挂载单个文件到容器中

```shell
$ docker run --rm -it \
# -v $HOME/.bash_history:/root/.bash_hisory \
--mount type=bind,source=$HOME/.bash_history,target=/root/.bash_history \
ubuntu:17.10 \
bash
# 这样就可以记录在容器输入过的命令了
```

