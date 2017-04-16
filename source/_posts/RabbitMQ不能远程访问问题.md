---
title: RabbitMQ不能远程访问问题
date: 2017-01-24 21:25:00
categories: [RabbitMQ]
tags: [RabbitMQ, 技术人生]
---

### 闲话 ###
&emsp;&emsp;本文讲述RabbitMQ不能远程访问的解决方法。

### 问题 ###
RabbitMQ不能远程访问，比如在node1机器上安装并启动RabbitMQ，但是在node2机器上写连接node1机器的RabbitMQ的代码，却连接不上。

### 解决 ###

#### 添加用户并授权 ####
rabbitmqctl add_user csh csh
rabbitmqctl set_user_tags csh administrator
rabbitmqctl set_permissions -p / csh ".*" ".*" ".*"

#### 配置 ####
在/etc/rabbitmq/rabbitmq.config中配置
```
[
  {rabbit,
    [
      {tcp_listeners, [5672]},
      {loopback_users, ["csh"]}
    ]
  }
].
```

#### 验证 ####
在node2中运行连接node1机器的RabbitMQ的代码