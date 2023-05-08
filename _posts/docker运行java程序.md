---
title: docker运行java程序
excerpt: 以war包为例,跑个java程序
date: 2021-02-05
categories: 容器化技术
tags: [docker, docker入门]
---



## 一 : 运行 war 程序

`hello-1.0.war` 的需要依赖 tomcat 才能提供服务, 所以定制镜像的时候需要 tomcat 作为基础镜像

编写 Dockerfile

```dockerfile
# 基础镜像
FROM tomcat:7.0.88-jre8
# 作者
MAINTAINER allen <allen@163.com>
# 定义环境变量
ENV TOMCAT_BASE /usr/local/tomcat
# 复制war包
COPY ./hello.war $TOMCAT_BASE/webapps/
```

开始构建

```shell
$ docker build -t hello:1.0 .
```

查看现有镜像

```shell
$ docker images
REPOSITORY			TAG				IMAGE ID			CREATED				SIZE
hello				1.0			dff88057b552		51 seconds ago		469 MB
```

运行 : 将宿主机的8888端口和容器的8080端口进行映射

```shell
$ docker run --name hello -d -p 8888:8080 hello:1.0
```

测试 : 查看服务器8888端口是否被监听

```shell
[yuelu@localhost sell]$ sudo netstat -anpl | grep 8888
tcp6       0      0 :::8888                 :::*                    LISTEN      9321/docker-proxy-c
```

浏览器中访问 : http://ip:8888/



## 二 : 运行 jar 程序

编写 Dockerfile 文件, 内容如下

```dockerfile
# 基础镜像
FROM openjdk:8-jdk-alpine
# 作者
MAINTAINER achelous<info4z@163.com>
# 复制jar包
COPY ./hello-1.0.jar hello.jar
# 执行命令
ENTRYPOINT ["java","-jar","hello.jar"]
```

执行构建

```shell
docker build -t hello:1.0 .
```

查看现有镜像

```shell
$ docker images
REPOSITORY                         TAG       IMAGE ID       CREATED          SIZE
hello                              1.0       843aef1a3352   19 seconds ago   122MB
```

启动容器

```shell
$ docker run -d -p 8080:8080 --name hello hello:1.0
```

查看容器

```shell
$ docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                       NAMES
9bda35727c12   hello:1.0   "java -jar hello.jar"    4 seconds ago   Up 3 seconds   0.0.0.0:8080->8080/tcp, :::8080->8080/tcp   hello
```

查看端口号

```shell
$ netstat -tnpl | grep 8080
(No info could be read for "-p": geteuid()=1000 but you should be root.)
tcp        0      0 0.0.0.0:8080            0.0.0.0:*               LISTEN      -                   
tcp6       0      0 :::8080                 :::*                    LISTEN      -
```

访问 : http://ip:8080/

