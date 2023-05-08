---
title: docker常用命令
excerpt: 熟悉一下docker命令
date: 2021-01-22
categories: 容器化技术
tags: [docker, docker入门]
---



## 一 : docker 镜像操作

docker 运行容器前需要本地存在对应的镜像; 如果本地不存在该镜像, docker 会从镜像仓库下载该镜像

### (一) 查找镜像

这个命令不常用, 通常习惯直接在[dockerhub](https://hub.docker.com/)上搜索

```yaml
Usage:  docker search [OPTIONS] TERM
Search Docker Hub for images
```

命令示例

```shell
$ docker search ubuntu
```

### (二) 下载镜像

从 docker 镜像仓库获取镜像的命令是 `docker pull`; 其命令格式为 : 

```yaml
Usage:  docker image pull [OPTIONS] NAME[:TAG|@DIGEST]
Download an image from a registry
Aliases:
  docker image pull, docker pull
```

具体的选项可用通过 `docker pull --help` 命令看到, 这里我们说一下镜像名称的格式;

1. docker 镜像仓库地址 : 地址的格式一般是 `<域名/IP>[:端口号]` ; 默认地址是 Docker Hub
2. 仓库名 : 这里的仓库名是两段式名称, 即 `<用户名>/<软件名>` ; 对于 docker hub, 如果不给出用户名, 则默认为 library, 也就是官方镜像

命令示例

```shell
$ docker pull ubuntu:16.04
# 命令中没有给出docker镜像仓库地址, 因此将会从docker hub获取镜像
# 镜像名称是ubuntu:16.04,因此将会从官方镜像library/ubuntu仓库中标签为16.04的镜像
```

### (三) 运行镜像

有了镜像后, 我们就能够以这个镜像为基础启动并运行一个容器; 语法 : 

```yaml
Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
Create and run a new container from an image
Aliases:
  docker container run, docker run
# 这个命令是容器操作命令,这里先简单提一下
```

以上面的 ubuntu:16.04 为例, 如果我们打算启动里面的 bash 并且进行交互式操作的话, 可用执行下面的命令

```shell
$ docker run -it --rm ubuntu:16.04 bash
# -it: 这是两个参数,一个是-i,交互式操作,一个是`-t`终端
# --rm: 这个参数是说容器退出后随之将其删除
# ubuntu:16.04: 这是指用ubuntu:16.04镜像为基础来启动容器
# bash: 放在镜像名后的是命令,这里我们希望有个交互式 shell,因此用的是bash
root@28ad92b30c8f:/# exit
# 通过exit退出这个容器
```

### (四) 列出镜像

要想列出已经下载下来的镜像, 可以使用 `docker image ls` 命令; 列表包含了**仓库名**, **标签**, **镜像ID**, **创建时间**以及**所占用的空间**

```shell
$ docker image ls
```

查看镜像, 容器, 数据卷所占用的空间

```shell
$ docker system df
```

仓库名, 标签均为 `<none> ` 的镜像称为虚悬镜像(dangling image), 显示这类镜像

```shell
$ docker image ls -f dangling=true
```

一般来说, 虚悬镜像已经失去了存在的价值, 是可以随意删除的, 可以用下面的命令删除

```shell
$ docker image prune
```

### (五) 删除本地镜像

如果要删除本地的镜像, 可以使用 `docker image rm` 命令, 其格式为 : 

```yaml
Usage:  docker image rm [OPTIONS] IMAGE [IMAGE...]
Remove one or more images
Aliases:
  docker image rm, docker image remove, docker rmi
```

使用 `docker image ls -q` 来配合 `docker image rm`, 这样可以批量删除希望删除的镜像

```shell
# 删除所有镜像名为 ubuntu 的镜像
$ docker image rm $(docker image ls -q ubuntu)
```

或者删除所有在 ubuntu:16.04 之前的镜像

```shell
$ docker image rm $(docker image ls -q -f before=ubuntu:16.04)
```

### (六) 导出与导入

导出语法

```yaml
Usage:  docker image save [OPTIONS] IMAGE [IMAGE...]
Save one or more images to a tar archive (streamed to STDOUT by default)
Aliases:
  docker image save, docker save
```

导入语法

```yaml
Usage:  docker image load [OPTIONS]
Load an image from a tar archive or STDIN
Aliases:
  docker image load, docker load
```

示例

```shell
# 导出镜像
$ docker save imageID > /目录/文件名.tar
# 导入镜像
$ docker load < 文件名.tar
```



## 二 : docker 容器操作

容器是独立运行的一个或一组应用, 以及它们的运行态环境; 对应的, 虚拟机可以理解为模拟运行的一整套操作系统(提供了运行态环境和其他系统环境)和跑在上面的应用

### (一) 启动容器

启动容器有两种方式, 一种是基于镜像新建一个容器并启动, 另外一个是将终止状态(stopped)的容器重新启动; 因为 docker 的容器实是轻量级的, 用户可以随时删除和新创建容器

**1: 新建并启动**

```yaml
Usage:  docker container run [OPTIONS] IMAGE [COMMAND] [ARG...]
Create and run a new container from an image
Aliases:
  docker container run, docker run
```

输出一个 "hello world", 之后终止容器

```shell
$ docker run ubuntu:16.04 /bin/echo 'Hello World'
```

**2: 启动已终止容器**

```yaml
Usage:  docker container start [OPTIONS] CONTAINER [CONTAINER...]
Start one or more stopped containers
Aliases:
  docker container start, docker start
```

启动一个 bash 终端, 允许用户进行交互

```shell
$ docker run -t -i ubuntu:16.04 /bin/bash
# -t: 让docker分配一个伪终端并绑定到容器的标准输入上
# -i: 让容器的标准输入保持打开
```

**3: 创建容器时在后台运行的标准操作**

| 步骤 | 操作内容                                               |
| ---- | ------------------------------------------------------ |
| 1    | 检查本地是否存在指定的镜像, 不存在就从公有仓库下载     |
| 2    | 利用镜像创建并启动一个容器                             |
| 3    | 分配一个文件系统, 并在只读的镜像层外面挂载一层可读写层 |
| 4    | 从宿主机配置的网桥接一个虚拟接口到容器中去             |
| 5    | 从地址池配置一个 ip 地址给容器                         |
| 6    | 执行用户指定的应用程序                                 |
| 7    | 执行完毕后容器被终止                                   |

### (二) 后台运行

很多时候, 需要让 docker 在后台运行儿不是直接把执行命令的结果输出在当前宿主机下

此时, 可以通过添加 `-d` 参数来实现

```shell
$ docker run -d hello-world
# 如果不使用-d参数运行容器,会把日志打印在控制台
# 如果使用了-d参数运行容器,不会输出日志,只会打印容器id(输出结果可以用docker logs查看)
```

**注意 :** 后台运行不是长久运行, 容器是否会长久运行, 是和 docker run 指定的命令有关, 和 `-d` 参数无关

### (三) 查看容器

查看运行中容器

```shell
$ docker container ls
```

查看所有容器

```shell
$ docker container ls -a
```

### (四) 停止运行的容器

可以使用 `docker container stop` 来终止一个运行中的容器

```yaml
Usage:  docker container stop [OPTIONS] CONTAINER [CONTAINER...]
Stop one or more running containers
Aliases:
  docker container stop, docker stop
```

处于终止状态的容器, 可以通过 `docker container start` 命令来重新启动

```shell
Usage:  docker container start [OPTIONS] CONTAINER [CONTAINER...]
Start one or more stopped containers
Aliases:
  docker container start, docker start
```

此外, `docker container restart` 命令会将一个运行态的容器终止, 然后再重新启动它

```yaml
Usage:  docker container restart [OPTIONS] CONTAINER [CONTAINER...]
Restart one or more containers
Aliases:
  docker container restart, docker restart
```

### (五) 进入容器

在使用 `-d` 参数时, 容器启动后会进入后台, 某些时候需要进入容器进行操作, 使用 `docker exec` 命令可以进入到运行中

```yaml
Usage:  docker container exec [OPTIONS] CONTAINER COMMAND [ARG...]
Execute a command in a running container
Aliases:
  docker container exec, docker exec
```

示例

```shell
docker exec -it 容器ID /bin/bash
# docker exec后边可以跟多个参数,这里主要说明-i和-t参数
# 只用-i参数时,由于没有分配伪终端,界面没有我们熟悉的linux命令提示符,但命令执行结果仍然可以返回
# 当-i和-t参数一起使用时,则可以看到我们熟悉的 linux 命令提示符
```

### (六) 导出和导入容器

**1: 导出容器**

```shell
# 如果要导出本地某个容器,可以使用docker export命令
docker export 容器ID > 导出文件名.tar
```

**2: 导入容器**

```shell
# 可以使用 docker import从容器快照文件中再导入为镜像
docker import [选项] file|URL|- [REPOSITORY[:TAG]]
# 或者
cat 导出文件名.tar | docker import - 镜像用户/镜像名:镜像版本
# 也可以通过指定 URL 或者某个目录来导入
docker import http://xxx.xxx.com/image.tgz- example/imagerepo
```

### (七) 删除容器

**1: 删除容器**

语法

```yaml
Usage:  docker container rm [OPTIONS] CONTAINER [CONTAINER...]
Remove one or more containers
Aliases:
  docker container rm, docker container remove, docker rm
```

示例

```shell
# 可以使用docker container rm来删除一个处于终止状态的容器
$ docker container rm ubuntu:16.04
# 如果要删除一个运行中的容器,可以添加-f参数; docker会发送SIGKILL信号给容器
```

**2: 清理所有处于终止状态的容器**

```shell
$ docker container prune
```

