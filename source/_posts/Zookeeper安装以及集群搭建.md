---
title: Zookeeper安装以及集群搭建
date: 2016-03-27 08:24:42
categories: [ZooKeeper]
tags: [ZooKeeper, 技术人生]
---
本文中配置3个节点的zookeeper集群，主机分别是node1，node2，node3
到官网下载压缩包，也可以在下面链接下载
[zookeeper-3.4.8.tar.gz](https://1csh1.github.io/file/Zookeeper安装以及集群搭建/zookeeper-3.4.8.tar.gz)

1.解压压缩包

```
tar -xvf zookeeper-3.4.8.tar.gz
```

2.修改配置
到conf文件目录下，有个zoo_sample.cfg文件，将文件拷贝一份改名为zoo.cfg

```
cp zoo_sample.cfg zoo.cfg
```

修改配置文件zoo.cfg

```
tickTime=2000
initLimit=10
syncLimit=5
#配置zookeeper的数据存放目录
dataDir=/csh/hadoop/zookeeper/data
#配置zookeeper的日志记录
dataLogDir=/csh/hadoop/zookeeper/datalog
clientPort=2181
#配置集群
server.1=node1:2888:3888
server.2=node2:2888:3888
server.3=node3:2888:3888
```

3.创建dataDir和dataLogDir目录

```
mkdir -p /csh/hadoop/zookeeper/data
mkdir -p /csh/hadoop/zookeeper/datalog
```

4.根据配置文件zoo.cfg中的集群，在dataDir中添加文件myid，并写入相应的数字

```
#在node1中执行
echo "1" > /csh/hadoop/zookeeper/data/myid
#在node2中执行
echo "2" > /csh/hadoop/zookeeper/data/myid
#在node3中执行
echo "3" > /csh/hadoop/zookeeper/data/myid
```

5.运行3个主机的zookeeper，通过以下命令

```
#在zookeeper/bin目录下
./zkServer.sh start
```

6.检测是否成功

```
#结点node1
[root@node1 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /csh/software/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower

#结点node2
[root@node2 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /csh/software/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: leader

#结点node3
[root@node3 bin]# ./zkServer.sh status
ZooKeeper JMX enabled by default
Using config: /csh/software/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower
```


