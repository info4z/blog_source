---
title: docker私有仓库
excerpt: 私有镜像通常不会对外开放,所以我们需要维护一个属于自己的私有仓库
date: 2021-02-19
categories: 容器化技术
tags: [docker, docker入门]
---



## 一 : Docker Hub

目前 Docker 官方维护了一个公共仓库 Docker Hub , 其中已经包括了数量超过 15000 的镜像; 大部分需求都可以通过在 Docker Hub 中直接下载镜像来实现

### (一) 注册登录

可以在 https://hub.docker.com 免费注册一个 Docker 账号

在命令行执行 `docker login` 输入用户名及密码来完成在命令行界面登录 Docker Hub; 可以通过 `docker logout` 退出登录

### (二) 拉取镜像

可以通过 `docker search` 命令来查找官方仓库中的镜像, 并利用 docker pull 命令将它下载到本地

### (三) 推送镜像

用户也可以在登陆后通过 `docker push` 命令来将自己的镜像推送到 `Docker Hub`

```shell
# 列出镜像
$ docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
hello-world   latest    feb5d9fea6a5   18 months ago   13.3kB

# 重命名
$ docker tag feb5d9fea6a5 zhang/hello-world:latest

# 查看
$ docker images
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
hello-world        latest    feb5d9fea6a5   18 months ago   13.3kB
zhang/helloworld   latest    feb5d9fea6a5   18 months ago   13.3kB

# 推送镜像
$ docker push zhang/hello-world:latest
```



## 二 : 私有仓库

有时候使用 Docker Hub 这样的公共仓库可能不方便, 用户可以创建一个本地仓库供私人使用; 比如基于公司内部项目构建的镜像

docker-registry 是官方提供的工具, 可以用于构建私有的镜像仓库

### (一) 安装运行 docker-registry

可以通过过去官方 registry 镜像来运行; 默认情况下, 仓库会被创建在容器的 `/var/lib/registry` 目录下, 默认端口号 : 5000

可以通过 `-v` 参数来将镜像文件存放在本地的指定路径

```shell
$ docker run --name registry -d -p 5000:5000 --restart=always \
-v /opt/data/registry:/var/lib/registry registry
```

### (二) 上传 | 搜索 | 下载镜像

创建好私有仓库之后, 就可以使用 docker tag 来标记一个镜像, 然后推送它到仓库

先在本机查看已有的镜像

```shell
$ docker image ls
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
zhang/hello-world       latest              feb5d9fea6a5        8 months ago        13.3 kB
```

标记镜像

```shell
docker tag IMAGE[:TAG] [REGISTRY_HOST[:REGISTRY_PORT]/]REPOSITORY[:TAG]
```

使用 docker tag 将 zhang/hello-world:latest 这个镜像标记为 127.0.0.1:5000/zhang/hello-world:latest; 

```shell
$ docker tag zhang/hello-world:latest 127.0.0.1:5000/zhang/hello-world:latest
```

使用 `docker push` 上传标记的镜像

```shell
$ docker push 127.0.0.1:5000/zhang/hello-world:latest
```

用 curl 查看仓库中的镜像

```shell
curl 127.0.0.1:5000/v2/_catalog
```

如果可以看到如下信息, 表示镜像已经被成功上传了

```json
{
    "repositories":[
        "zhang/hello-world"
    ]
}
```

先删除已有镜像, 再尝试从私有仓库中下载这个镜像

```shell
$ docker image rm 127.0.0.1:5000/zhang/hello-world:latest
$ docker pull 127.0.0.1:5000/zhang/hello-world:latest
```

### (三) 私有仓库地址

如果不想使用 `127.0.0.1:5000` 作为仓库地址, 比如想让本网段的其他主机也能把镜像推送到私有仓库; 就得把例如 `10.0.0.20:5000` 这样的内网地址作为私有仓库地址, 这时会无法成功推送镜像

```shell
# 改名
$ docker tag hello-world:latest 10.0.0.20:5000/zhang/hello-world
# 校验结果
$ docker images
REPOSITORY                         TAG       IMAGE ID       CREATED         SIZE
registry                           latest    8db46f9d7550   7 hours ago     24.2MB
10.0.0.20:5000/zhang/hello-world   latest    feb5d9fea6a5   18 months ago   13.3kB
# 推到私有仓库
$ docker push 10.0.0.20:5000/zhang/hello-world
Using default tag: latest
The push refers to repository [10.0.0.20:5000/zhang/hello-world]
Get "https://10.0.0.20:5000/v2/": http: server gave HTTP response to HTTPS client
```

这是因为 Docker 默认不允许非 HTTPS 方式推送镜像; 我们可以通过 Docker 的配置选项来取消这个限制

对于使用 `systemd` 的系统(如: ubuntu 16.04+, Debian 8+, centos 7), 请在 `/etc/docker/daemon.json` 中写入如下内容(如果文件不存在则需要新建该文件); 对于 Docker for Windows, Docker for Mac 在设置中编辑 daemon.json 增加和上边一样的字符串即可

```json
{
    "registry-mirror":[
        "https://registry.docker-cn.com"
    ],
    "insecure-registries":[
        "192.168.100.100:5000"
    ]
}
```

重启docker之后再尝试

```shell
$ docker push 10.0.0.20:5000/zhang/hello-world
Using default tag: latest
The push refers to repository [10.0.0.20:5000/zhang/hello-world]
e07ee1baac5f: Pushed 
latest: digest: sha256:f54a58bc1aac5ea1a25d796ae155dc228b3f0e11d046ae276b39c4bf2f13d8c4 size: 525
```



