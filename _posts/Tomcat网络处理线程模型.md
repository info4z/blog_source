---
title: Tomcat网络处理线程模型
excerpt: 作为java项目最常用的web服务器,不能满足于简单使用
date: 2020-09-04
categories: 高性能编程
tags: [java系统性能调优, 性能调优]
---



## 一 : BIO + 同步 Servlet

一个请求，一个工作线程，CPU利用率低。

新版本中不再使用

```mermaid
graph LR
A[User Request] --> B[Nginx] --> C[BIO Connector] --> D[Servlet]
D[Servlet] --> E1[Remote Service]
D[Servlet] --RPC--> E2[Remote Service]
D[Servlet] --HTTP--> E3[Remote Service]

subgraph Nginx
B[Nginx]
end

subgraph Tomcat
C[BIO Connector]
D[Servlet]
end

subgraph Remote
E1
E2
E3
end

subgraph API Gateway
Nginx
Tomcat
end
```



## 二 : APR + 异步 Servlet

APR(Apache Portable Runtime/Apache可移植运行库)，是Apache HTTP服务器的支持库

JNI(Java Native Interface)的形式调用Apache HTTP服务器的核心动态链接库来处理文件读取或网络传输操作

Tomcat默认监听指定路径，如果有APR安装，则自动启用

```mermaid
graph LR
A[User Request] --HTTP/1.1--> B[Nginx] --> C[APR Connector] --> D[Async Servlet]
D[Async Servlet] --> E1[Remote Service]
D[Async Servlet] --Async RPC--> E2[Remote Service]
D[Async Servlet] --Async HTTP--> E3[Remote Service]

subgraph Nginx
B
end

subgraph Tomcat
C
D
end

subgraph Remote
E1
E2
E3
end

subgraph API Gateway
Nginx
Tomcat
end
```

## 三 : NIO + 异步 Servlet

Tomcat8开始，默认NIO方式

非阻塞读取请求信息，非阻塞处理下一个请求，完全异步

```mermaid
graph LR
A[User Request] --HTTP/1.1--> B[Nginx] --> C[NIO Connector] --> D[Async Servlet]
D[Async Servlet] --Async RPC--> E1[Remote Service]
D[Async Servlet] --Async HTTP--> E2[Remote Service]
D[Async Servlet] --> E3[Remote Service]

subgraph Nginx
B
end

subgraph Tomcat
C
D
end

subgraph Remote
E1
E2
E3
end

subgraph API Gateway
Nginx
Tomcat
end
```

## 四 : NIO 处理流程

```mermaid
graph TD
A[User Request] --1--> B[Acceptor]
B --3--> C1[PollerEvent]
B --3--> C2[PollerEvent]
C1 --4--> D1[Poller1 selector1]
C2 --4--> D2[Poller2 selector2]
D1 --5--> E1
D2 --5--> E1
E1 --> F1[nioChannel1]

subgraph C[PollerEvent]
C1[PollerEvent]
C2[PollerEvent]
end

subgraph D[Poller selector]
D1[Poller1 selector1]
D2[Poller2 selector2]
end

subgraph E[SocketProcessor]
E1[SocketProcessor1]
E2[SocketProcessor2]
E3[SocketProcessor3]
end

subgraph F[nioChannel]
F1
F2[nioChannel2]
F3[nioChannel3]
end

F1 --2--> B
```

1. 接受器接受套接字
2. 接受器从缓存中检索niochannel对象
3. Pollerthread将nioChannel注册到它的选择器I0事件
4. 轮询器将nioChannel分配给一个work线程来处理请求
5. SocketProcessor完成对请求的处理和返回

### 