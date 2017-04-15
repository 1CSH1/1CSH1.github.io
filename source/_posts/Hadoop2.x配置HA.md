---
title: Hadoop2.x配置HA
date: 2016-03-27 10:53:11
categories: [Hadoop]
tags: [Hadoop2.x, 技术人生, HA]
---
各节点配置参考表

| 主机 | NameNode | DataNode | Zookeeper | ZKFC | JournalNode | ResourceManager | NodeManager |
|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|:-----:|
|node1|1||1|1||1||
|node2|1|1|1|1|1||1|
|node3||1|1||1||1|
|node4||1|||1||1|


文件配置：
**core-site.xml**

```
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/csh/hadoop/hadoop2.7.2/tmp</value>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://mycluster</value>
    </property>
    <property>
        <name>dfs.journalnode.edits.dir</name>
        <value>/csh/hadoop/hadoop2.7.2/journal</value>
    </property>
    <property>
        <name>ha.zookeeper.quorum</name>
        <value>node1:2181,node2:2181,node3:2181</value>
    </property>
```

**hdfs-site.xml**

```
    <property>
        <name>dfs.nameservices</name>
        <value>mycluster</value>
    </property>
    <property>
        <name>dfs.ha.namenodes.mycluster</name>
        <value>nn1,nn2</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn1</name>
        <value>node1:8020</value>
    </property>
    <property>
        <name>dfs.namenode.rpc-address.mycluster.nn2</name>
        <value>node2:8020</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.mycluster.nn1</name>
        <value>node1:50070</value>
    </property>
    <property>
        <name>dfs.namenode.http-address.mycluster.nn2</name>
        <value>node2:50070</value>
    </property>
    <property>
        <name>dfs.namenode.shared.edits.dir</name>
        <value>qjournal://node2:8485;node3:8485;node4:8485/mycluster</value>
    </property>
    <property>
        <name>dfs.client.failover.proxy.provider.mycluster</name>
        <value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
    </property>
    <property>
        <name>dfs.ha.fencing.methods</name>
        <value>sshfence</value>
    </property>
    <property>
        <name>dfs.ha.fencing.ssh.private-key-files</name>
        <value>/root/.ssh/id_dsa</value>
    </property>
    <property>
        <name>dfs.ha.fencing.ssh.connect-timeout</name>
        <value>30000</value>
    </property>
```

**mapred-site.xml**

```
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
```

**yarn-site.xml**

```
   <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>node1</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
         <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
         <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
```

**masters**

```
node2
```

**slaves**

```
node2
node3
node4
```

启动

安装Zookeeper请看：[Zookeeper安装以及集群搭建](https://1csh1.github.io/2016/03/27/Zookeeper%E5%AE%89%E8%A3%85%E4%BB%A5%E5%8F%8A%E9%9B%86%E7%BE%A4%E6%90%AD%E5%BB%BA/)

1.启动 zookeeper（在node1，node2，node3中执行以下命令）
(在zookeeper/bin目录下)

```
./zkServer.sh start
```

通过以下命令检查是否启动成功

```
./zkServer.sh status
```

成功会显示以下数据

```
ZooKeeper JMX enabled by default
Using config: /csh/software/zookeeper-3.4.8/bin/../conf/zoo.cfg
Mode: follower //这里会有一个节点是：leader，其余2个节点是：follower
```


2.启动journalnode（在node1中执行以下命令）

```
./hadoop-daemons.sh start journalnode
```

在node2、node3、node4运行jps命令检查journalnode是否启动成功
成功会有出现

```
2601 JournalNode
```



3.格式化zkfc,让在zookeeper中生成ha节点（在node1中执行）

```
hdfs zkfc –formatZK
```

格式化成功后可以查看zookeeper得到

```
./zkCli.sh -server node1:2181
[zk: node1:2181(CONNECTED) 0] ls /hadoop-ha
[mycluster]

```

4.格式化hdfs（在node1中执行）

```
hadoop namenode –format
```


5.启动NameNode
先在node1上启动active结点（在node1中执行）

```
[root@node1 sbin]# hadoop-daemon.sh start namenode
```

在node2中同步namenode数据，同时启动standby的namenode

```
#把NameNode的数据同步到node2上  
hdfs namenode –bootstrapStandby  
#启动node2上的namenode作为standby  
hadoop-daemon.sh start namenode 
```


6.启动DataNode（在node1中执行）

```
./hadoop-daemons.sh start datanode
```

7.启动yarn
（在作为资源管理器上的机器上启动，我这里是node1,执行如下命令完成yarn的启动）

```
./start-yarn.sh
```


8.启动ZKFC（在node1、node2中分别执行）

```
hadoop-daemon.sh start zkfc 
```


各节点的情况

```
//node1
17827 QuorumPeerMain
18179 NameNode
25431 Jps
19195 ResourceManager
19985 DFSZKFailoverController

//node2
9088 QuorumPeerMain
13250 Jps
9171 JournalNode
10360 NodeManager
10985 DFSZKFailoverController
9310 NameNode
9950 DataNode

//node3
7108 NodeManager
7926 Jps
6952 DataNode
6699 JournalNode
6622 QuorumPeerMain

//node4
6337 JournalNode
6755 NodeManager
7574 Jps
6603 DataNode
```
