---
title: HBase完全分布式配置
date: 2016-04-02 22:53:26
categories: [HBase]
tags: [HBase, 技术人生]
---

 | Node Name  | Master | ZooKeeper | RegionServer | 
  |:--------:|:--------:|:--------:|:--------:|  
 |node1 | yes | yes | no | 
 | node2 |  backup | yes | yes | 
 | node3 | no |  yes | yes | 

1.配置conf/regionservers

```
node2
node3
```

2.创建conf/backup-masters

```
node2
```

3.配置 hbase-site.xml

```
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://node1:9000/hbase</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/csh/hadoop/zookeeper</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
        <description>指定HBase的运行模式：false：单机模式或者伪分布模式； true：完全分布式模式</description>
    </property>
    <property>
        <name>hbase.master</name>
        <value>hdfs://node1:60000</value>
        <description>指定master位置</description>
    </property>
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>node1,node2,node3</value>
    </property>
```

4.将配置文件拷贝到node2、node3中

```
scp /csh/software/hbase-1.1.4/conf/* root@node2:/csh/software/hbase-1.1.4/conf/
scp /csh/software/hbase-1.1.4/conf/* root@node3:/csh/software/hbase-1.1.4/conf/
```

5.启动HBase（在node1启动）

```
./start-hbase.sh
```

6.jps查看结果

```
//node1
4339 Jps
4122 HMaster
4047 HQuorumPeer

//node2
3313 HQuorumPeer
3810 Jps
3447 HMaster
3374 HRegionServer

//node3
2981 HQuorumPeer
3048 HRegionServer
3230 Jps
```

