---
title: RabbitMQ消息可靠性和插件化机制
excerpt: 可靠性内容不多,顺便记录几个插件吧...
date: 2020-11-27
categories: 中间件
tags: [分布式消息中间, RabbitMQ]
---



# 一 : 消息可靠性

RabbitMQ的消息可靠性, 一般是业务系统接入消息中间件时首要考虑的问题,一般通过三个方面保障

| 可靠性     | 描述                                  |
| ---------- | ------------------------------------- |
| 发送可靠性 | 确保消息成功发送到 Broker             |
| 存储可靠性 | Broker 对消息持久化, 确保消息不会丢失 |
| 消费可靠性 | 确保消息成功被消费                    |

## (一) 消息发送可靠性

—般消息发送可靠性分为三个层级

| 层级          | 描述                                           |
| ------------- | ---------------------------------------------- |
| At most once  | 最多一次, 消息可能会丢失, 但绝不会重复传输     |
| At least once | 最少一次, 消息绝不会丢失, 但可能会重复传输     |
| Exactly once  | 恰好一次, 每条消息肯定会被传输一次且仅传输一次 |

RabbitMQ支持其中的**最多一次**和**最少一次**; 

* **最少一次**投递实现需要考虑以下这个几个方面的内容:
* 消息生产者需要开启事务机制或者 publisher confirm 机制, 以确保消息可以可靠地传输到 RabbitMQ 中
  * 消息生产者需要配合使用mandatory参数或者备份交换器来确保消息能够从交换器路由到队列中, 进而能够保存下来而不会被丢弃
  
* **最多一次**的方式就无须考虑以上那些方面, 生产者随意发送, 不过这样很难确保消息会成功发送。




## (二) 消息消费可靠性

消费者在消费消息的同时, 需要将 autoAck 设置为false, 然后通过手动确认的方式去确认己经正确消费的消息,以免在消费端引起不必要的消息丢失。

# 二 : 插件机制

## (一) 概述

RabbitMQ支持插件, 通过插件可以扩展多种核心功能:支持多种协议、系统状态监控、其它AMQP 0-9-1交换类型、节点联合等。许多功能都是通过插件实现的。

RabbitMQ内置一些插件,通过命令可以查看插件列表。

```sh
rabbitmq-plugins list
```

## (二) 启用插件

通过rabbitmq-plugins命令可以启用或禁用插件

```sh
rabbitmq-plugins enable plugin-name
rabbitmq-plugins disable plugin-name
```

## (三) 常用插件

| 插件                        | 作用                                                         |
| --------------------------- | ------------------------------------------------------------ |
| rabbitmq_auth_mechanism_ssl | 身份验证机制插件, 允许RabbitMQ客户端使用x509证书和TLS(PKI)证书进行身份验证 |
| rabbitmq_event_exchange     | 事件分发插件, 使客户端可以接收到Broker 的 queue.deleted, exchange.created, binding.created 等事件 |
| rabbitmq_management         | 基于Web界面的管理/监控插件                                   |
| rabbitmq_management_agent   | 启用 rabbitmq_management 时, 会自动启用此插件, 用于在 Web 管理中查看集群节点 |
| rabbitmq_mqtt               | MQTT 插件,使 RabbitMQ 支持 MQTT 协议                         |
| rabbitmq_web_mqtt           | 使 RabbitMQ 支持通过 WebSocket 订阅消息, 基于 MQTT 协议传输  |





