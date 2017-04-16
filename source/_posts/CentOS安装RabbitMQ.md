---
title: CentOS安装RabbitMQ
date: 2017-01-24 21:22:00
categories: [RabbitMQ, Linux]
tags: [CentOS, RabbitMQ, 技术人生]
---

### 闲话 ###
&emsp;&emsp;本文讲述在CentOS操作系统安装RabbitMQ的过程。

### Erlang ###

下载Erlang：
[erlang-18.1-1.el7.centos.x86_64.rpm](http://download.csdn.net/detail/u011642663/9743498)

安装Erlang：
```
rpm -ivh erlang-18.1-1.el7.centos.x86_64.rpm
```

### RabbitMQ ###

下载RabbitMQ：
[rabbitmq-server-3.6.6-1.el7.noarch.rpm](http://download.csdn.net/detail/u011642663/9743499)

安装RabbitMQ：
```
rpm -ivh rabbitmq-server-3.6.6-1.el7.noarch.rpm
```

### /etc/profile ###

在/etc/profile配置文件中设置
```
ERLANG_HOME=/usr/lib64/erlang
RABBITMQ_HOME=/usr/lib/rabbitmq

PATH=$PATH:$ERLANG_HOME/bin:$RABBITMQ_HOME/bin

export PATH ERLANG_HOME RABBITMQ_HOME
```

执行下面命令更新系统参数配置
```
source /etc/profile
```

### 验证 ###
执行下面命令
```
rabbitmq-server
```

在另一个终端中检查状态
```
rabbitmqctl status
```

如果没问题说明安装成功