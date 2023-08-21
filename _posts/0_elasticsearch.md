---
title: elasticsearch
date: 2023-03-24
excerpt: 记录一下安装步骤
categories: 服务器
tags: [es]
---



## 一、概述

官网：https://www.elastic.co/cn/elasticsearch/

The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash（也称为 ELK Stack）。能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。

Elaticsearch，简称为 ES，ES 是一个开源的高扩展的分布式全文搜索引擎，是整个 Elastic Stack 技术栈的核心。它可以近乎实时的存储、检索数据；本身扩展性很好，可以扩展到上百台服务器，处理 PB 级别的数据。

目前市面上流行的搜索引擎软件，主流的就两款：Elasticsearch 和 Solr。这两款都是基于 Lucene 搭建的，可以独立部署启动的搜索引擎服务软件。由于内核相同，所以两者除了服务器安装、部署、管理、集群以外，对于数据的操作 修改、添加、保存、查询等等都十分类似。

选择 ES 的原因：

1. 易于使用：一个下载和一个命令就可以启动一切
2. 分析查询：如果除了搜索文本之外还需要它来处理分析查询，ES 是更好的选择
3. 分布式：如果需要分布式索引，则需要选择 ES 。对于需要良好可伸缩性和以及性能分布式环境，ES 是更好的选择
4. 日志：ES 在开源日志管理用例中占据主导地位，许多组织在 ES 中索引它们的日志以使其可搜索
5. 监控和指标：相对于 Solr，ES 暴露了更多的关键指标

## 二、安装

下载解压

```shell
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.10-x86_64.rpm
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.10-x86_64.rpm.sha512
shasum -a 512 -c elasticsearch-7.17.12-x86_64.rpm.sha512 
```

安装

```shell
rpm --install elasticsearch-7.17.12-x86_64.rpm
```

启动

```shell
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
```

监听所有IP

```shell
vim /etc/elasticsearch/elasticsearch.yml

# 集群名称
cluster.name: elasticsearch
# 节点名称
node.name: node-1
# 监听地址
network.host: 0.0.0.0
# 监听端口
http.port: 9200
# 集群初始化节点(单机部署不需要)
cluster.initial_master_nodes: ["node-1"]
```

重启

```shell
systemctl restart elasticsearch
```

## 三、设置密码

编辑配置文件

```shell
vim /etc/elasticsearch/elasticsearch.yml
# 开启安全配置
xpack.security.enabled: true
# 单机需要加上这个
discovery.type: single-node
```

设置密码

```shell
rpm -ql elasticsearch-7.17.10-1.x86_64 | grep elasticsearch-setup-passwords
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```

说明

```yaml
elastic: 内置超级用户
apm_system: 
kibana_system: kibana连接es,仅在kibana.yml中使用
logstash_system: logstash在es中存储监控信息使用,仅在logstash配置文件中使用
beats_system: 
remote_monitoring_user: 
```

修改密码(这里用的是kibana中的开发工具窗口)

```shell
POST _security/user/elastic/_password
{"password":"222222"}
```

## 四、kibana

安装

```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.12-x86_64.rpm
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.17.12-x86_64.rpm.sha512
shasum -a 512 -c kibana-7.17.12-x86_64.rpm.sha512 
sudo rpm --install kibana-7.17.12-x86_64.rpm
```

编辑配置文件

```shell
vim config/kibana.yml
# 默认端口
server.port: 5601
# ES 服务器的地址
elasticsearch.hosts: ["http://localhost:9200"]
# 索引名
kibana.index: ".kibana"
# 支持中文
i18n.locale: "zh-CN"
# 账号/密码
elasticsearch.username: "kibana_system"
elasticsearch.password: "Zhang@123"
```

明文密码不安全，可以采用 keystore 形式

```shell
# 也可以用keystore形式存储密码
bash kibana-keystore create
bash kibana-keystore add elasticsearch.password
systemctl restart kibana
```



