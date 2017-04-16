---
title: HBase单机模式配置
date: 2016-04-02 22:43:26
categories: [HBase]
tags: [HBase, 技术人生]
---
1.在hbase-env.sh中修改Java路径

```
export JAVA_HOME=/csh/link/jdk
export HBASE_PID_DIR=/csh/hadoop/hbase/pids
```

2.在hbase-site.xml中修改
   

```
    <property>
        <name>hbase.rootdir</name>
        <value>file:///csh/hadoop/hbase</value>
    </property>
    <property>
        <name>hbase.zookeeper.property.dataDir</name>
        <value>/csh/hadoop/zookeeper</value>
    </property>
```

3.启动HBase

```
bin/start-hbase.sh
```

4.连接HBase

```
bin/hbase shell
```

