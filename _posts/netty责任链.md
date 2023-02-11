---
title: netty责任链
date: 2020-07-24
categories:
- 高性能编程
tags: 
- 高并发网络编程
- Netty
---



> netty的责任链称为pipeline, 类似流水线作业, 一定要结合源码, 多看几遍



## 一 : 设计模式 - 责任链模式

责任链模式(Chain of Responsibility Pattern)为请求创建了一个处理对象的链。
**发起请求和具体处理请求的过程进行解耦** : 职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可, 无须关心请求的处理细节和请求的传递

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230207212922203.png)

### (一) : 实现责任链模式

实现责任链模式4个要素 : 

1. 处理器抽象类
2. 具体的处理器实现类
3. 保存处理器信息
4. 处理执行

伪代码示例

```java
// 集合形式存储: 类似tomcat中filters
// 1.处理器抽象类
class AbstractHandler { 
    void doHandler(Object arg0); 
}

// 2.处理器具体实现类
class Handler1 extends AbstractHandler { assert coutinue; }
class Handler2 extends AbstractHandler { assert coutinue; }
class Handler3 extends AbstractHandler { assert coutinue; }

// 3.创建集合并存储所有处理器实例信息
List handlers = new List();
handlers.add(handler1, handler2, handler3);

// 4.处理请求，调用处理器()
void Process(request){
    for( handler in handlers){
        handler.doHandler(request) ;
    }
}
// 发起请求调用，通过责任链处理请求
call.process(request);
```

```java
//链表形式调用: 参考netty的实现形式
// 1.处理器抽象类
class AbstractHandler {
    AbstractHandle next;//下一个节点
    void doHandler (Object argo); // handler方法
}
// 2.处理器具体实现类
class Handler1 extends AbstractHandler { assert coutinue; }
class Handler2 extends AbstractHandler { assert coutinue; }
class Handler3 extends AbstractHandler { assert coutinue;}

// 3.将处理器串成链表存储
// pipeline=头[ handler1 -> handler2 ->handler3]尾

// 4.处理请求，调用处理器(从头到尾)
void Process( request){
    handler = pipeline.findOne;
    //查找第一个
    while(hand != null){
        handler.doHandler(request);
        handler = handler.next();
    }
}
```



### (二) : Netty 中的 ChannelPipeline 责任链

**Pipeline管道**保存了通道所有处理器信息。

创建新channel时自动创建一个专有的pipeline。入站**事件**和出站**操作**会调用pipeline上的处理器

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230207215047975.png) 



## 二 : 事件

### (一) 入站事件和出站事件

**入站事件 :** 通常指I/O线程生成了入站数据。(通俗理解:从socket底层自己往上冒上来的事件都是入站)

比如EventLoop收到selector的OP_READ事件，入站处理器调用socketChannel.read(ByteBuffer)接收到数据后，这将导致通道的ChannelPipeline中包含的下一个中的channelRead方法被调用。

**出站事件 :** 经常是指I/O线程执行实际的输出操作。(通俗理解:想主动往socket底层操作的事件的都是出站)

比如bind方法用意是请求server socket绑定到给定的SocketAddress，这将导致通道的ChannelPipeline中包含的下一个出站处理器中的bind方法被调用。

### (二) Netty 中事件的定义

入站事件(inbound)

| 事件                          | 描述                  |
| ----------------------------- | --------------------- |
| fireChannelRegistered         | channel注册事件       |
| fireChannelUnregistered       | channel解除注册事件   |
| fireChannelActive             | channel活跃事件       |
| fireChannelInactive           | channel非活跃事件     |
| fireExceptionCaught           | 异常事件              |
| fireUserEventTriggered        | 用户自定义事件        |
| fireChannelRead               | channel读事件         |
| fireChannelReadComplete       | channel读完成事件     |
| fireChannelWritabilityChanged | channel写状态变化事件 |

出站事件(outbound)

| 事件          | 描述                              |
| ------------- | --------------------------------- |
| bind          | 端口绑定事件                      |
| connect       | 连接事件                          |
| disconnect    | 断开连接事件                      |
| close         | 关闭事件                          |
| deregister    | 解除注册事件                      |
| flush         | 刷新数据到网络事件                |
| read          | 读事件，用于注册OP_READ到selector |
| write         | 写事件                            |
| writeAndFlush | 写出数据事件                      |



## 三 : 处理器

### (一) Pipeline 中的 handler 是什么

**ChannelHandler :** 用于处理 I/O 事件或拦截 I/O 操作, 并转发到 ChannelPipeline 中的下一个处理器; 这个顶级接口定义功能很弱, 实际使用时会去实现以下两大子接口 : 处理入站 I/O 事件的 ChannelInBoundHandler, 处理出站 I/O 操作的 ChannelOutBoundHandler

**适配器类 :** 为了开发方便, 避免所有 handler 去实现一遍接口方法, Netty 提供了简单的实现类

- 处理入站 I/O 事件 : ChannelInboundHandlerAdapter
- 处理出站 I/O 事件 : ChannelOutboundHandlerAdapter
- 同时处理入站和出站事件 : ChannelDuplexHandler

**ChannelHandlerContext :** 实际存储在 Pipeline 中的对象并非 ChannelHandler, 而是上下文对象; 将 handler 包裹在上下文对象中, 通过上下文对象与它所属的 ChannelPipeline 交互, 向上或向下传递事件或者修改 pipeline 都是通过上下文对象

### (二) 维护 Pipeline 中的 handler

ChannelPipeline是线程安全的，ChannelHandler可以在任何时候添加或删除。

例如，你可以在即将交换敏感信息时插入加密处理程序，并在交换后删除它。

一般操作，初始化的时候增加进去，较少删除。

Pipeline 中管理 handler 的API

| 方法名称    | 描述                 |
| ----------- | -------------------- |
| addFirst    | 最前面插入           |
| addLast     | 最后面插入           |
| addBefore   | 插入到指定处理器前面 |
| addAfter    | 插入到指定处理器后面 |
| remove      | 移除指定处理器       |
| removeFirst | 移除第一个处理器     |
| removeLast  | 移除最后一个处理器   |
| replace     | 替换指定的处理器     |


伪代码示例

```java
ChannelPipeline p = ...;
p.addLast("1", new InboundHandlerA());
p.addLast("2", new InboundHandlerB());
p.addLast("3", new OutboundHandlerA());
p.addLast("4", new OutboundHandlerB());
p.addLast("5", new InboundOutboundHandlerX()); // 聚合处理器
```

### (三) handler 的执行分析

按之前伪代码逻辑, 现在的责任链如图所示

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230210151919349.png) 

由此可以推断 : 

* 当入站事件时, 执行顺序是 : 1 => 2 => 3 => 4 => 5
* 当出站事件时, 执行顺序是 : 5 => 4 => 3 => 2 => 1

在这一原则之上, ChannelPipeline在执行时会进行选择3和4为出站处理器, 因此, 实际执行是 :

* 入站事件的执行顺序是 1 => 2 => 5, 1和2为入站处理器
* 出站事件的执行顺序是 5 => 4 => 3

不同的入站事件会触发handler不同的方法执行 : 

* 上下文对象中 `fire**` 开头的方法, 代表**入站**事件传播和处理

* 其余的方法代表**出站**事件的传播和处理。



## 四 : 分析

### (一) 分析 registered 入站事件的处理

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211135536424.png) 

ServerSocketChannel.pipeline的变化

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211135748415.png) 

### (二) 分析 bind 出站事件的处理

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211140021280.png) 

### (三) 分析 accept 入站事件的处理

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211140848456.png) 

这是一个分配的过程，main Group负责accept，然后分配sub Group负责read

### (四) 分析 read 入站事件的处理

pipeline分析的关键4要素:什么事件、有哪些处理器、哪些会被触发、执行顺序

![](https://gcore.jsdelivr.net/gh/info4z/blog_images@main/images/image-20230211141208378.png) 



## 五 : 小结

用户在管道中有一个或多个 channelhandler 来接收 I/O 事件(例如读取)和请求 I/O 操作(例如写入和关闭)

一个典型的服务器在每个通道的管道中都有以下处理程序, 但是根据协议和业务逻辑的复杂性和特征, 可能会有所不同

* 协议解码器 : 将二进制数据(例如 ByteBuf)转换为 Java 对象

* 协议编码器 : 将 Java 对象转换为二进制数据

* 业务逻辑处理程序 : 执行实际的业务逻辑(例如数据库访问)

