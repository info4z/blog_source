---
title: docker私有镜像
excerpt: 利用Dockerfile构建私有镜像
date: 2021-01-29
categories: 容器化技术
tags: [docker, docker入门]
---



## 一 : Dockerfile入门

镜像的定制实际上就是定制每一层所添加的配置, 文件; 我们可以把每一层修改, 安装, 构建, 操作的命令都写入一个脚本, 这个脚本就是 Dockerfile; **Dockerfile 是一个文本文件, 其内包含了一条条的指令, 每一条指令构建一层, 因此每一条指令的内容, 就是描述该层应当如何构建**。

入门案例 : 以官方 nginx 镜像为例, 使用 Dockerfile 来定制

```shell
# 在一个空白目录中, 建立一个文本文件, 并命名为 Dockerfile
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```

这个 Dockerfile 很简单, 一共就两行; 涉及到了两条指令, **FROM** 和 **RUN**

```dockerfile
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

构建镜像, 不要忘记最后的 `.`

```shell
# 在 Dockerfile 文件所在目录执行
$ docker build -t nginx:v1.0 .
```

查看镜像

```shell
$ docker image ls
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
nginx         v1.0      64e674311108   3 minutes ago   142MB
```

### (一) FROM 指定基础镜像

所谓定制镜像, 一定是以一个镜像为基础, 在其上进行定制; 基础镜像是必须指定的, 而 FROM 就是指定基础镜像, 因此一个 Dockerfile 中 FROM 是**必备的指令**, 并且**必须是第一条**指令。

在 Docker Hub 上有非常多的高质量的官方镜像, 有可以直接拿来使用的服务类的镜像, 如 nginx, redis, mysql, tomcat 等; 可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制; 如果没有找到对应服务的镜像, 官方镜像中还提供了一些更为基础的操作系统镜像, 如 ubuntu, debian, centos, alpine 等, 这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外, Docker 还存在一个特殊的镜像, 名为 scratch; 这个镜像是虚拟的概念, 并不实际存在, 它表示一个**空白的镜像**

```dockerfile
FROM scratch
...
```

如果你以 scratch 为基础镜像的话, 意味着你不以任何镜像为基础, 接下来缩写的指令将作为镜像的第一层开始存在。

对于 linux 下静态编译的程序来说, 并不需要有操作系统提供运行时支持, 所需的一切库都已经在可执行文件里了, 因此直接 FROM scratch 会让镜像体积更加小巧; 使用 Go 语言开发的应用很多会使用这种方式来制作镜像, 这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。

### (二) RUN 执行命令

RUN 指令是用来执行命令行命令的; 由于命令行的强大能力, RUN 指令在定制镜像时是**最常用**的指令之一; 其格式有两种 : 

```dockerfile
# 1.shell格式: RUN <命令>
RUN echo `<h1>Hello,Docker!</h1>` > /usr/share/nginx/html/index.html
# 2.exec格式: RUN ["可执行文件","参数1","参数2"]
RUN ["./test.php", "dev", "offline"]	# 等价于 RUN ./test.php dev offline
```

**注意**：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层, 会造成镜像膨胀过大。例如：

```dockerfile
FROM centos
RUN yum install -y wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
```

以上执行会创建 3 层镜像。可简化为以下格式：

```dockerfile
FROM centos
RUN yum -y install wget \
	&& wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
	&& tar -xvf redis.tar.gz
```

如上, 以 `&&` 符号连接命令, 这样执行后, 只会创建 1 层镜像。



## 二 : 构建镜像

语法

```yaml
Usage:	docker build [OPTIONS] PATH | URL | -
Build an image from a Dockerfile
```

示例

```shell
$ docker build -t nginx:v1.0 .
Sending build context to Docker daemon 2.048 kB
Step 1/2 : FROM nginx
Trying to pull repository docker.io/library/nginx ... 
latest: Pulling from docker.io/library/nginx
f1f26f570256: Pull complete 
7f7f30930c6b: Pull complete 
2836b727df80: Pull complete 
e1eeb0f1c06b: Pull complete 
86b2457cc2b0: Pull complete 
9862f2ee2e8c: Pull complete 
Digest: sha256:2ab30d6ac53580a6db8b657abf0f68d75360ff5cc1670a85acb5bd85ba1b19c0
Status: Downloaded newer image for docker.io/nginx:latest
 ---> 080ed0ed8312
Step 2/2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in a541be57c6df

 ---> 8d8b91b5def2
Removing intermediate container a541be57c6df
Successfully built 8d8b91b5def2
# 从命令的输出结果中, 我们可以清晰的看到镜像的构建过程; 
# 在Step2中,RUN指令启动了一个容器a541be57c6df,执行了指定命令,并提交了这一层8d8b91b5def2,随后删除了a541be57c6df容器
```

注 : 指令最后的那个`.` 表示**上下文路径**, 是指 docker 在构建镜像, 有时候想要使用到本机的文件(比如复制), docker build 命令得知这个路径后, 会将路径下的所有内容打包。因此, 上下文路径下不要放无用的文件, 因为会一起打包发送给 docker 引擎, 如果文件过多会造成过程缓慢。



## 三 : Dockerfile 指令详解

### (一) COPY

COPY 指令将从构建上下文目录中 <源路径> 的文件或目录**复制**到新的一层的镜像内的 <目标路径> 位置; <目标路径> 是容器内的指定路径, 该路径不用事先建好, 路径不存在的话, 会自动创建。

语法格式

```dockerfile
COPY [--chown=<user>:<group>] <源路径1>...  <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",...  "<目标路径>"]
# [--chown=<user>:<group>]: 可选参数,用户改变复制到容器内文件的拥有者和属组
```

示例

```dockerfile
COPY package.json /usr/src/app/
```

<源路径> 可以是多个, 甚至可以是通配符, 如 : 

```dockerfile
COPY hom* /mydir/
COPY hom?.txt /mydir/
```

### (二) ADD

**更高级的复制文件**, ADD 指令和 COPY 的格式和性质基本一致; 但是在 COPY 基础上增加了一下功能。

比如 <源路径> 可以是一个 URL, 这种情况下, docker 引擎会试图去下载这个链接的文件放到 <目标路径> 去。

在 docker 官方的 Dockerfile 最佳实践文档中要求, **尽可能的使用 COPY**, 因为 COPY 的语义很明确, 就是复制文件而已, 而 ADD 则包含了更复杂的功能, 其行为也不一定很清晰; 最适合使用 ADD 的场合, 就是所提及的需要自动解压缩的场合; 因此, 在 COPY 和 ADD 指令中选择的时候, 可以遵循这样的原则, **所有的文件复制均使用 COPY 指令, 仅在需要自动解压缩的场合使用 ADD**

### (三) CMD

为启动的容器**指定默认要运行的程序和参数**, 程序运行结束, 容器也就结束。CMD 指令指定的程序可被 docker run 命令行参数中指定要运行的程序所覆盖。

格式 

```dockerfile
CMD <shell 命令> 
CMD ["<可执行文件或命令>","<param1>","<param2>",...] # 推荐,执行过程中第一种在会自动转换为这种,且默认可执行文件是sh
CMD ["<param1>","<param2>",...]  # 该写法是为ENTRYPOINT指令指定的程序提供默认参数
```

类似于 RUN 指令, 用于运行程序, 但二者运行的时间点不同

- CMD 在docker run 时运行
- RUN 是在 docker build 时运行

**注意 :** 如果 Dockerfile 中如果存在多个 CMD 指令, 仅最后一个生效。

### (四) ENTRYPOINT

和 CMD 一样, 都是在**指定容器启动程序及参数**, 但其不会被 docker run 的命令行参数指定的指令所覆盖, 而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。但是, 如果 docker run 时使用了 `--entrypoint` 选项, 将覆盖 ENTRYPOINT 指令指定的程序。

格式

```dockerfile
ENTRYPOINT ["<executeable>","<param1>","<param2>",...]
```

**优点 :** 在执行 docker run 的时候可以指定 ENTRYPOINT 运行所需的参数。

**注意 :** 如果 Dockerfile 中如果存在多个 ENTRYPOINT 指令, 仅最后一个生效。

可以搭配 CMD 命令使用：一般是变参才会使用 CMD , 这里的 CMD 等于是在给 ENTRYPOINT 传参

```dockerfile
# 构建镜像 nginx:test
FROM nginx

ENTRYPOINT ["nginx", "-c"] # 定参
CMD ["/etc/nginx/nginx.conf"] # 变参 
```

示例

```sh
# 不传参运行
$ docker run nginx:test		# 相当于容器启动进程 nginx -c /etc/nginx/nginx.conf
# 传参运行
$ docker run nginx:test -c /etc/nginx/new.conf	# # 相当于容器启动进程 nginx -c /etc/nginx/new.conf
```

### (五) ENV

**设置环境变量**, 定义了环境变量, 那么在后续的指令中, 就可以使用这个环境变量。

格式

```dockerfile
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
```

示例

```dockerfile
ENV NODE_VERSION 7.2.0

RUN curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.xz" \
	&& curl -SLO "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc"
```

### (六) ARG

**构建参数**, 与 ENV 作用一致。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效, 也就是说只有 docker build 的过程中有效, 构建好的镜像内不存在此环境变量。但是不要因此就使用 ARG 保存密码之类的信息, 因为 docker history 还是可以看到所有值的

格式

```dockerfile
ARG <参数名>[=<默认值>]
```

**注意 :** 构建命令 docker build 中可以用 `--build-arg <参数名>=<值>` 来覆盖。

### (七) VOLUME

**定义匿名数据卷**。在启动容器时忘记挂载数据卷, 会自动挂载到匿名卷。

格式

```dockerfile
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
```

作用：

- 避免重要的数据, 因容器重启而丢失, 这是非常致命的。
- 避免容器不断变大。

**注意 :** 在启动容器 docker run 的时候, 我们可以通过 -v 参数修改挂载点。

容器运行时应该尽量保存容器存储层不发生写操作, 对于数据库类需要保存动态数据看应用, 其数据库文件应该保存于卷(volume)中, 为了防止运行时用户忘记将动态文件所保存目录挂载为卷, Dockerfile 中, 我们可以实现指定某些目录挂载为匿名卷, 这样在运行时如果用户不指定挂载, 其应用也可以正常运行, 不会向容器存储层写入大量数据

```dockerfile
# 这里的/data目录就会在运行时自动挂载为匿名卷,任何向/data中写入的信息都不会记录进容器存储层,从而保证了容器存储层的无状态化
VOLUME /data
```

当然, 运行时可以覆盖这个挂载设置, 比如

```shell
docker run -d -v mydata:/data IMAGE
# 在这行命令中,就是用了mydata这个命名卷挂载到了/data这个位置,替代了Dockerfile中定义的匿名卷的挂载配置
```

### (八) EXPOSE

仅仅只是**声明端口**, 在运行时并不会因为这个声明应用就会开启这个端口的服务

格式

```dockerfile
EXPOSE <端口1> [<端口2>...]
```

作用：

- 帮助镜像使用者理解这个镜像服务的守护端口, 以方便配置映射。
- 在运行时使用随机端口映射时, 也就是 `docker run -P` 时, 会自动随机映射 EXPOSE 的端口。

### (九) WORKDIR

**指定工作目录**。用 WORKDIR 指定的工作目录, 会在构建镜像的每一层中都存在。以后各层的当前目录就被改为指定的目录, 如该目录不存在, WORKDIR 会帮你建立目录。

格式

```dockerfile
WORKDIR <工作目录路径>
```

docker build 构建镜像过程中的, 每一个 RUN 命令都是新建的一层。只有通过 WORKDIR 创建的目录才会一直存在。

这里说一个重点: 不要把 Dockerfile 等同于 Shell 脚本来书写, 这种错误的理解还可能会导致出现下面这样的错误

```dockerfile
RUN cd /app
RUN echo "hello" > world.txt
# 如果将这个Dockerfile进行构建镜像运行后,会发现找不到/app/world.txt文件
```

原因分析 : 
* 在 Shell 中, 连续两行是同一个进程执行环境, 因此前一个命令修改的内存状态, 会直接影响后一个命令
* 而在 Dockerfile 中, 这两行 RUN 命令的执行环境根本不同, 是两个完全不同的容器

这就是对 Dockerfile 构建分层存储的概念不了解导致的错误
* 每一个 RUN 都是启动一个容器, 执行命令, 然后提交存储层文件变更
* 第一层 `RUN cd /app` 的执行仅仅是当前进程的工作目录变更, 一个内存上的变化而已, 其结果不会造成任何文件变更; 而到第二层的时候, 启动的是一个全新的容器, 跟第一层的容器更完全没关系, 自然不可能继承前一层构建过程中的内存变化
* 因此如果需要改变以后各层的工作目录的位置, 那么应该使用 `WORKDIR` 指令

### (十) USER

用于指定**执行后续命令的用户和用户组**, 这边只是切换后续命令执行的用户(用户和用户组必须提前已经存在)。

格式

```dockerfile
USER <用户名>[:<用户组>]
```

示例

```dockerfile
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN ["redis-server"]
```

### (十一) HEALTHCHECK

**健康检查**, 用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

格式 

```dockerfile
HEALTHCHECK [选项] CMD <命令>	# 设置检查容器健康状况的命令
HEALTHCHECK NONE				# 如果基础镜像有健康检查指令,使用这行可以屏蔽掉其健康检查指令

HEALTHCHECK [选项] CMD <命令>	# 这边CMD后面跟随的命令使用,可以参考CMD的用法。
```

**HEALTHCHECK** 指令是告诉 Docker 应该如何进行判断容器的状态是否正常, 这是 Docker 1.12 引入的新指令; 通过该指令指定一行命令, 用这个命令来判断容器主进程的服务状态是否还正常, 从而比较真是的反应容器实际状态

一个镜像制定了 HEALTHCHECK 指令之后, 用其启动容器, 初始状态会为 starting, 在执行健康检查成功后变为 healthy, 如果连续一定次数失败, 则会变为 unhealthy

HEALTHCHECK 支持下列选项

| 选项              | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| --interval=<间隔> | 两次健康检查的间隔, 默认为30秒                               |
| --timeout=<时长>  | 健康检查命令运行超时时间, 如果超过这个时间, 本次健康检查就被视为失败, 默认 30 秒 |
| --retries=<次数>  | 当连续失败指定次数后, 则将容器状态是为 unhealthy, 默认 3 次  |

为了帮助排障, 健康检查命令的输出(包括stdout以及stderr)都会被存储于健康状态里, 可以用 `docker inspect` 来查看

```sh
# Return low-level information on Docker objects
$ docker inspect
```

### (十二) ONBUILD

用于**延迟构建**命令的执行。简单的说, 就是 Dockerfile 里用 ONBUILD 指定的命令, 在本次构建镜像的过程中不会执行(假设镜像为  test-build)。当有新的 Dockerfile 使用了之前构建的镜像 `FROM test-build` , 这时执行新镜像的  Dockerfile 构建时候, 会执行 test-build 的 Dockerfile 里的 ONBUILD 指定的命令。

格式

```dockerfile
ONBUILD <其它指令>
```

Dockerfile 中的其他指令都是为了定制当前镜像而准备的, 唯有 ONBUILD 是为了帮着别人定制自己而准备的



## 四 : 其他制作镜像方式

Docker 还提供了 docker load 和 docker save 命令, 用以将镜像保持为一个 tar 文件, 然后传输到另一个位置上, 再加载进来; 

这是在没有 Docker Registry 时的做法, 现在已经不推荐, 镜像迁移应该直接使用 Docker Registry, 无论是直接使用 Docker Hub 还是是使用内网私有 Registry 都可以

例如 : 保存 nginx 镜像

```shell
$ docker save nginx | gzip > nginx-latest.tar.gz
```

然后我们将 nginx-latest.tar.gz 文件复制到了另一个机器上, 再次加载镜像

```shell
$ docker load -i nginx-latest.tar.gz
```

