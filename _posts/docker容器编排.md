---
title: docker容器编排
excerpt: Dockerfile可以定义单独的镜像,而Compose可以对多个服务进行统一的镜像制作和容器编排
date: 2021-03-05
categories: 容器化技术
tags: [docker, docker进阶]
---



## 一 : Compose 简介

Compose 项目是 Docker 官方的开源项目, 负责实现对 Docker 容器集群的快速编排; 其代码目前在 [github](https://github.com/docker/compose) 上开源。

**Compose 定位** 是 **定义和运行多个 Docker 容器的应用**, 其前身是开源项目 Fig。

我们指定使用一个 Dockerfile 模板文件, 可以让用户很方便的定义一个单独的应用容器; 然而, 在日常工作中, 经常会碰到需要多个容器相互配合来完成某些任务的情况; 例如要实现一个 web 项目, 除了 **web** 服务容器本身, 往往还需要再加上后端的**数据库**服务容器, 甚至还包括**负载均衡**容器等; Compose 恰好满足了这样的需求; 它允许用户通过一个单独的 docker-compose.yml 模板文件来定义一组相关联的应用容器为一个项目(project)。

Compose 中有**两个重要的概念**。
* 服务 (service) : 一个应用的容器, 实际上可以包括若干运行相同镜像的容器实例
* 项目 (project) : 由一组关联的应用容器组成的一个完整业务单元

总结一下
* **Compose 的默认管理对象是项目**, 通过子命令对项目中的一组容器进行便捷的生命周期管理
* **Compose 项目**由 Python 编写, 实现上调用了 Docker 服务提供的 API 来对容器进行管理



## 二 : Compose 安装

Compose 支持 Linux, MacOS, Windows 10 三大平台; Compose 可以通过 Python 的包管理工具 pip 进行安装, 也可以直接下载编译好的二进制文件使用, 甚至能够直接在 Docker 容器中运行。

Docker for Mac, Docker for Windows 自带 docker-compose 二进制文件, 安装 Docker 之后可以直接使用; Linux 系统需要单独使用二进制或者 pip 方式(ARM架构,如树莓派)进行安装。

**二进制包**在 Linux 上的安装十分简单, 从官方 GitHub Release 处直接下载编译好的二进制文件即可; 例如 : 在 Linux 64 位系统上直接下载对应的二进制包。

```shell
# 下载
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 赋予可执行权限
$ sudo chmod +x /usr/local/bin/docker-compose
```

校验是否安装成功

```sh
$ docker-compose --version
```



## 三 : Compose 使用

一个项目可以由多个服务(容器)关联而成, Compose **面向项目**进行管理。

**场景 :** 最常见的项目是 web 网站, 一般的 web 网站都会依赖第三方提供的服务(比如: DB 和 cache)。

### (一) 代码准备

maven依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

配置文件

```yaml
spring:
  profiles:
    active: uat
---
spring:
  profiles: uat
  redis:
    host: 127.0.0.1
---
spring:
  profiles: pro
  redis:
    host: redis
```

代码 : 模拟访问量统计(使用 redis)

```java
@RestController
@SpringBootApplication
public class WebApplication {

    @Autowired
    private StringRedisTemplate redisTemplate;

    @GetMapping
    public String stats() {
        redisTemplate.opsForValue().setIfAbsent("num", "0");
        redisTemplate.opsForValue().increment("num");
        return "当前访问量: " + redisTemplate.opsForValue().get("num");
    }
    
    public static void main(String[] args) {
        SpringApplication.run(WebApplication.class, args);
    }
}
```

使用 maven 进行编辑打包

```shell
$ mvn clean package -Dmaven.test.skip=true
```

### (二) Dockerfile

在 dubbo-admin 目录下编写 Dockerfile 文件

```dockerfile
# 表示使用jdk8环境作为集成镜像
FROM openjdk:8-jdk-alpine
# 作者
MAINTAINER zhang<info4z@163.com>
# 匿名数据卷
VOLUME /tmp/web
# 拷贝文件并重命名
ADD ./web-1.0.jar web.jar
# 为了缩短Tomcat的启动时间,添加java.security.egd的系统属性指向/dev/urandom作为ENTRYPOINT
ENTRYPOINT ["java","-jar","web.jar","--spring.profiles.active=pro"]
```

构建

```shell
$ docker build -t web:1.0 .
```

### (三) docker-compose.yml

在项目根目录下编写 docker-compose.yml 文件, 这个是 Compose 使用的主模板文件

```yaml
version: '3'

networks:
	overlay:

services:
	web:
		build: .
		ports:
			- 80:8080
		networks:
			- overlay
	redis:
		image: redis:latest
		ports:
			- 6379:6379
		networks:
			- overlay
```

### (四) 运行 compose

在 docker-compose.yml 文件所在目录执行

```sh
# 正常启动
$ docker-compose up
# 后台启动
$ docker-compose up -d
```

在浏览器中访问 `http://ip:80` 进行验证



## 四 : Compose 命令说明

docker-compose 命令的基本的使用格式是

```yaml
docker compose [OPTIONS] COMMAND
OPTIONS: 
	-f, --file stringArray: 指定模板文件,默认为docker-compose.yml,可以多次指定
	-p, --project-name string: 指定项目名称, 默认将使用所在目录名称作为项目名
```

### (一) 镜像操作

| 命令   | 说明                                       |
| ------ | ------------------------------------------ |
| images | List images used by the created containers |
| pull   | Pull service images                        |
| push   | Push service images                        |

### (二) 容器操作

| 命令    | 说明                                     |
| ------- | ---------------------------------------- |
| ps      | List containers                          |
| up      | Create and start containers              |
| restart | Restart service containers               |
| down    | Stop and remove containers, networks     |
| kill    | Force stop service containers            |
| rm      | Removes stopped service containers       |
| exec    | Execute a command in a running container |
| events  | Receive real time events from containers |

### (三) 服务操作

| 命令    | 说明                               |
| ------- | ---------------------------------- |
| create  | Creates containers for a service   |
| build   | Build or rebuild services          |
| start   | Start services                     |
| stop    | Stop services                      |
| pause   | Pause services                     |
| unpause | Unpause services                   |
| run     | Run a one-off command on a service |

### (四) 其他

| 命令    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| config  | Parse, resolve and render compose file in canonical format   |
| cp      | Copy files/folders between a service container and the local filesystem |
| logs    | View output from containers                                  |
| ls      | List running compose projects                                |
| port    | Print the public port for a port binding                     |
| top     | Display the running processes                                |
| version | Show the Docker Compose version information                  |



## 五 : Compose 模板文件

Compose文件是一个 yaml 文件，用于定义[服务](https://dockerdocs.cn/compose/compose-file/compose-file-v3/index.html#service-configuration-reference), [网络](https://dockerdocs.cn/compose/compose-file/compose-file-v3/index.html#network-configuration-reference)和[卷](https://dockerdocs.cn/compose/compose-file/compose-file-v3/index.html#volume-configuration-reference)。撰写文件的默认路径为`./docker-compose.yml`。

| 顶层结构 | 作用       |
| -------- | ---------- |
| version  | 声明版本   |
| services | 定义服务   |
| networks | 定义网络   |
| volumes  | 定义数据卷 |

指令关键字比较多, 但大部分指令跟 docker run 相关参数的含义都是类似的, 具体可以查询 [官方文档](https://docs.docker.com/compose/compose-file/compose-file-v3/) 或者 [中文文档](https://dockerdocs.cn/compose/compose-file/compose-file-v3/)。

示例

```yaml
version: "3.9"				# 声明版本

networks:					# 定义网络: 一个前端,一个后端
  frontend:
  backend:

volumes:					# 定义数据卷
  db-data:

services:					# 定义服务
  redis:					# 服务名:redis
    image: redis:alpine		# 镜像
    ports:					# 端口
      - "6379"
    networks:				# 连接网络
      - frontend
    deploy:					# 部署
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:					# 数据卷
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        max_replicas_per_node: 1
        constraints:
          - "node.role==manager"
```

每个服务都必须通过 image 指令指定镜像或 build 指令(需要 Dockerfile)等来自动构建生成镜像; 如果使用 build 指令, 在 Dockerfile 中设置的选项(如: CMD, EXPOSE, VOLUME, ENV等)将会自动被获取, 无需在 docker-compose.yml 中再次设置

