---
title: 用Haproxy来做RabbitMQ集群的负载均衡
date: 2017-01-24 21:32:00
categories: [RabbitMQ]
tags: [RabbitMQ, 技术人生]
---

### 闲话 ###
&emsp;&emsp;本文讲述在RabbitMQ集群的基础下，用Haproxy来做负载均衡，node6,node7,node8这3台机器已经组成一个RabbitMQ集群了,在node9机器上配置Haproxy来做负载均衡。

### 配置HAProxy ###

#### 下载 ####
[haproxy-1.7.1.tar.gz](http://download.csdn.net/detail/u011642663/9743582)

#### 解压 ####
```
tar -xzvf haproxy-1.7.1.tar.gz
```

#### 编译 ####
```
cd haproxy-1.7.1
make TARGET=generic
```

编译完目录下有haproxy可执行文件
将haproxy复制到/usr/local/sbin
```
cp haproxy /usr/local/sbin
```

#### 配置文件 ####
```
vi config_file
//输入以下配置
listen rabbitmq_cluster
    bind node9:5670
	mode tcp 
	timeout client 3h
	timeout server 3h
	timeout connect 3h
	balance roundrobin
	server rabbit6 node6:5672 check inter 5000 rise 2 fall 3
	server rabbit7 node7:5672 check inter 5000 rise 2 fall 3
	server rabbit8 node8:5672 check inter 5000 rise 2 fall 3
	
listen private_monitoring 
    bind node9:8100
	mode http
	option httplog
	timeout client 3h
    timeout server 3h
    timeout connect 3h
	stats enable
	stats uri /stats
	stats refresh 5s
```

#### 启动Haproxy ####
```
haproxy -f config_file
```

#### Haproxy自带的配置文件 ####
```
haproxy-1.7.1/doc/configuration.txt
```