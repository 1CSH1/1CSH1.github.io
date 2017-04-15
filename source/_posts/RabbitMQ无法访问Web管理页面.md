---
title: RabbitMQ无法访问Web管理页面
date: 2017-01-24 21:28:00
categories: [RabbitMQ]
tags: [RabbitMQ, 技术人生]
---

### 闲话 ###
&emsp;&emsp;本文讲述解决RabbitMQ启动后没法访问Web管理页面的问题。

### 问题 ###
启动RabbitMQ后，没法访问Web管理页面

### 解决 ###
RabbitMQ安装后默认是不启动管理模块的，所以需要配置将管理模块启动
启动管理模块命令如下
```
rabbitmqctl start_app
rabbitmq-plugins enable rabbitmq_management
rabbitmqctl stop
```

### 验证 ###
如果你是安装在本机器则访问http://localhost:15672/，发现可以访问Web管理页面了