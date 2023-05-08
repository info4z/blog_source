---
title: RabbitMQ入门
excerpt: 记录RabbitMQ的安装过程和简单使用
date: 2020-10-30
categories: 中间件
tags: [分布式消息中间, RabbitMQ]
---



## 一 : 简介

RabbitMQ 是一个开源的 AMQP 实现, 服务器端用 Erlang 语言编写, 支持多种客户端。

用于在分布式系统中存储转发消息, 在易用性、扩展性、高可用性等方面表现不俗。

官方网站 : [https://www.rabbitmq.com/](https://www.rabbitmq.com/)



## 二 : 安装

环境准备 : ContOS 7, Erlang

```shell
# 下载erlang
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v21.3.8.21/erlang-21.3.8.21-1.el7.x86_64.rpm
# 安装erlang
rpm -ivh erlang-21.3.8.21-1.el7.x86_64.rpm
# 安装socat
yum install -y socat
# 下载 RabbitMQ
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.7.28/rabbitmq-server-3.7.28-1.el7.noarch.rpm
# 安装
rpm -ivh rabbitmq-server-3.7.28-1.el7.noarch.rpm
```

使用命令

```shell
# 启动
systemctl start rabbitmq-server
# 查看状态
systemctl status rabbitmq-server
# 开机启动
systemctl enable rabbitmq-server
```



## 三 : 配置

### (一) 配置文件

RabbitMQ 有一套默认的配置, 能够满足日常开发需求, 如果需要修改, 需要自己创建一个配置文件

```sh
touch /etc/rabbitmq/rabbitmq.conf
```

配置文件示例 : [https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example](https://github.com/rabbitmq/rabbitmq-server/blob/master/docs/rabbitmq.conf.example)

配置项说明 : [https://www.rabbitmq.com/configure.html#config-items](https://www.rabbitmq.com/configure.html#config-items)

### (二) 默认端口

RabbitMQ 会绑定一些端口, 安装完后, 需要将这些端口添加至防火墙

| 端口        | 描述                                                         |
| ----------- | ------------------------------------------------------------ |
| 4369        | 是 Erlang 的端口/结点名称映射程序, 用来跟踪节点名称监听地址, 在集群中起到一个类似DNS的作用 |
| 5672, 5671  | AMQP0-9-1 和 1.0 客户端端口, 没有使用 SSL 和使用 SSL 的端口  |
| 25672       | 用于 RabbitMQ 节点间和 CLI 工具通信, 配合 4369 使用          |
| **15672**   | HTTP_API 端口, 管理员用户才能访问, 用于管理 RabbitMQ, 需要启用management插件 |
| 61613,61614 | 当STOMP插件启用的时候打开, 作为STOMP客户端端口(根据是否使用 TLS 选择) |
| 1883,8883   | 当 MQTT 插件启用的时候打开, 作为 MQTT 客户端端口根据是否使用 TLS 选择) |
| 15674       | 基于 WebSocket 的STOMP客户端端口(当插件Web STOMP启用的时候打开) |
| 15675       | 基于 WebSocket 的 MQTT 客户端端口(当插件Web MQTT启用的时候打开) |

释放端口

```shell
# 释放端口
firewall-cmd --zone=public --add-port=4369/tcp --permanent
firewall-cmd --zone=public --add-port=5672/tcp --permanent
firewall-cmd --zone=public --add-port=25672/tcp --permanent
firewall-cmd --zone=public --add-port=15672/tcp --permanent
# 重启防火墙
firewall-cmd --reload
```



## 四 : 使用

### (一) 管理界面

RabbitMQ安装包中带有管理插件, 但需要手动激活

```sh
# 查看帮助命令
rabbitmq-plugins --help
# 列出所有插件
rabbitmq-plugins list
# 激活插件,因为有依赖关系,实际上是启动了3个
rabbitmq-plugins enable rabbitmq_management
```

访问 : http://ip:15672

```yaml
账号: guest
密码: guest
```

“guest”用户默认只能通过本机访问, 要让其它机器可以访问, 需要创建一个新用户, 为其分配权限

```sh
#添加用户,参数格式: username,password
rabbitmqctl add_user admin admin 
# 为用户分配角色,参数格式: username tag
rabbitmqctl set_user_tags admin administrator 
#为用户分配资源权限,参数格式: -p vhost username conf wirte read
rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
```

RabbitMQ 的用户角色(tags)分类 : none、management、policymaker、monitoring、administrator

| tag           | 作用                                                         |
| ------------- | ------------------------------------------------------------ |
| none          | 不能访问 management plugin                                   |
| management    | 1.用户可以通过 AMQP 做的任何事<br>2.列出自己可以通过 AMQP 登入的 virtual hosts<br>3.查看自己的 virtual hosts 中的 queues, exchanges 和 bindings<br>4.查看和关闭自己的 channels 和 connections<br>5.查看有关自己的 virtual hosts 的“全局”的统计信息, 包含其他用户在这些virtual hosts中的活动 |
| policymaker   | 1.management 可以做的任何事<br>2.查看、创建和删除自己的 virtual hosts 所属的 policies 和 parameters |
| monitoring    | 1.management可以做的任何事<br>2.列出所有virtual hosts, 包括他们不能登录的virtual hosts<br>3.查看其他用户的connections和channels<br>4.查看节点级别的数据如clustering和memory使用情况<br>5.查看真正的关于所有virtual hosts的全局的统计信息 |
| administrator | 1.policymaker和monitoring可以做的任何事<br>2.创建和删除virtual hosts<br>3.查看、创建和删除users<br>4.查看创建和删除permissions<br>5.关闭其他用户的connections |



### (二) java中使用

maven 依赖

```xml
<dependency>
    <groupld>com.rabbitmq</groupld>
    <artifactld>amqp-client</artifactld>
    <version>5.5.1</version>
</dependency>
```

简单队列生产者

```java
public class Producer {

    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接属性
        factory.setHost("192.168.100.242");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = null;
        Channel channel = null;

        try {
            // 3、从连接工厂获取连接
            connection = factory.newConnection("生产者");

            // 4、从链接中创建通道
            channel = connection.createChannel();

            /**
             * 5、声明(创建)队列,如果队列不存在,才会创建;不允许声明两个队列名相同,属性不同的队列,否则会报错
             *
             * 参数说明：
             * 1) queue: 队列名称
             * 2) durable: 队列是否持久化
             * 3) exclusive: 是否排他,即是否为私有的,如果为true,会对当前队列加锁,其它通道不能访问,并且在连接关闭时会自动删除,不受持久化和自动删除的属性控制;一般在队列和交换器绑定时使用
             * 4) autoDelete: 是否自动删除,当最后一个消费者断开连接之后是否自动删除
             * 5) arguments: 队列参数,设置队列的有效期、消息最大长度、队列中所有消息的生命周期等等
             */
            channel.queueDeclare("queue1", false, false, false, null);

            // 6、发送消息
            channel.basicPublish("", "queue1", null, "Hello World!".getBytes());
            
            // 7、关闭通道 关闭连接
            channel.close();
            connection.close();

        } catch (Exception e) {
            e.printStackTrace();
        } 
    }
}
```

简单队列消费者

```java
public class Consumer {

    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接属性
        factory.setHost("192.168.100.242");
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = null;
        Channel channel = null;

        try {
            // 3、从连接工厂获取连接
            connection = factory.newConnection("消费者");

            // 4、从链接中创建通道
            channel = connection.createChannel();

            // 5、声明(创建)队列
            channel.queueDeclare("queue1", false, false, false, null);

            // 6、定义收到消息后的回调
            DeliverCallback callback = new DeliverCallback() {
                public void handle(String consumerTag, Delivery message) throws IOException {
                    System.out.println("收到消息: " + new String(message.getBody(), "UTF-8"));
                }
            };
            // 7、监听队列
            channel.basicConsume("queue1", true, callback, new CancelCallback() {
                public void handle(String consumerTag) throws IOException {
                }
            });

            System.out.println("开始接收消息");
            System.in.read();
			// 8、关闭通道,关闭连接
            channel.close();
            connection.close();
            
        } catch (Exception e) {
            e.printStackTrace();
        } 
    }
}
```

### (二) spring中使用

spring 依赖

```xml
<dependency>
    <groupld>org.springframework.amqp</groupld>
    <artifactld>spring-amqp</artifactld>
    <version>2.1.1.RELEASE</version>
</dependency>
<dependency>
    <groupld>org.springframework.amqp</groupld>
    <artifactld>spring-rabbit</artifactld>
    <version>2.1.1.RELEASE</version>
</dependency>
```

springboot 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

配置文件

```yaml
spring:
  rabbitmq:
    host: 10.0.0.11
    port: 5672
    username: admin
    password: admin
```

启动类

```java
@SpringBootApplication
public class RabbitApplication {
    public static void main(String[] args) {
        SpringApplication.run(RabbitApplication.class, args);
    }
}
```

生产者

```java
@Component
public class Producer {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @Bean
    CommandLineRunner runner() {
        return args -> rabbitTemplate.convertAndSend("queue1", "hello spring");
    }
}
```

消费者

```java
@Component
public class Consumer {
    @RabbitListener(queues = "queue1")
    public void processMessage(String message){
        System.out.println(message);
    }
}
```

