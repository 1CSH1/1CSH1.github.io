---
title: 搭建RabbitMQ集群
date: 2017-01-24 21:32:00
categories: [RabbitMQ]
tags: [RabbitMQ, 技术人生]
---

### 闲话 ###
&emsp;&emsp;本文讲述搭建RabbitMQ集群，文中总共用了3个节点node6,node7,node8。

### 环境准备 ###
将node6,node7,node8的.erlang.cookie文件内容设置为一样的
```
//在node6节点上
cat /var/lib/rabbitmq/.erlang.cookie
//将内容复制到node7,node8节点上，可能需要修改文件的权限

//结果：比如我自己的机器上的.erlang.cookie文件内容
rabbit6  ETIEGPLEHVXLJDVZGLMF
rabbit7  ETIEGPLEHVXLJDVZGLMF
rabbit8  ETIEGPLEHVXLJDVZGLMF
```

### 启动RabbitMQ ###

在node6上运行
```
RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit6 \rabbitmq-server -detached
```

在node7上运行
```
RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit7 \rabbitmq-server -detached
```

在node8上运行
```
RABBITMQ_NODE_PORT=5672 RABBITMQ_NODENAME=rabbit8 \rabbitmq-server -detached
```


### 将node7节点加入到集群中 ###

在node7机器上运行如下命令
```
//停止rabbit7节点上的RabbitMQ应用程序
rabbitmqctl -n rabbit7@csh07 stop_app

//重设rabbit7节点的元数据和状态为清空
rabbitmqctl -n rabbit7@csh07 reset

//将rabbit7加入到rabbit6集群中
rabbitmqctl -n rabbit7@csh07 join_cluster rabbit6@csh06 \ rabbit7@csh07

//重新启动rabbit7节点的应用程序
rabbitmqctl -n rabbit7@csh07 start_app
```

### 将node8节点加入到集群中 ###

在node8机器上运行如下命令
```
rabbitmqctl -n rabbit8@csh08 stop_app
rabbitmqctl -n rabbit8@csh08 reset
rabbitmqctl -n rabbit8@csh08 join_cluster rabbit6@csh06
rabbitmqctl -n rabbit8@csh08 start_app
```

### 查看集群状态 ###
命令如下：
```
rabbitmqctl cluster_status
```

结果：
```
[root@csh08 ~]# rabbitmqctl -n rabbit8@csh08 cluster_status
Cluster status of node 'rabbit8@csh08' ...
[{nodes,[{disc,['rabbit7@csh07','rabbit6@csh06']},
         {ram,['rabbit8@csh08']}]},
 {running_nodes,['rabbit6@csh06','rabbit7@csh07',
                 'rabbit8@csh08']},
 {cluster_name,<<"rabbit6@csh06.itc.cmbchina.cn">>},
 {partitions,[]},
 {alarms,[{'rabbit6@csh06',[]},
          {'rabbit7@csh07',[]},
          {'rabbit8@csh08',[]}]}]
```


### 将节点从集群中移除 ###
命令如下：
```
rabbitmqctl -n rabbit8@csh08 stop_app
rabbitmqctl -n rabbit8@csh08 reset
rabbitmqctl -n rabbit8@csh08 start_app
```

### 可能遇到问题 ###
在添加节点node7到node6上时，遇到连接不上node6节点机器
```
[root@csh07 ~]# rabbitmqctl -n rabbit7@csh07 join_cluster rabbit6@csh06
Clustering node 'rabbit7@csh07' with 'rabbit6@csh06' ...
Error: unable to connect to nodes ['rabbit6@csh06']: nodedown

DIAGNOSTICS
===========

attempted to contact: ['rabbit6@csh06']

rabbit6@csh06:
  * unable to connect to epmd (port 4369) on csh06: nxdomain (non-existing domain)


current node details:
- node name: 'rabbitmq-cli-45@csh07'
- home dir: /var/lib/rabbitmq
- cookie hash: a5xQrcy5rMqVgP/ZCwWUPA==
```

解决：
在所有机器上的/etc/hosts文件加上如下配置，主要是添加csh06,csh07,csh08映射
```
192.168.1.6 node6 csh06
192.168.1.7 node7 csh07
192.168.1.8 node8 csh08
```