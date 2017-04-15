---
title: Sqoop安装与配置
date: 2016-04-19 18:59:25
categories: [Sqoop]
tags: [Sqoop, 技术人生]
---
### 下载 ###

[sqoop-1.4.6 安装包](https://1csh1.github.io/file/Sqoop安装与配置/sqoop-1.4.6.bin__hadoop-2.0.4-alpha.tar.gz)

### 配置 ###

sqoop-env.sh

```
#Set path to where bin/hadoop is available 配置Hadoop
export HADOOP_COMMON_HOME=/csh/link/hadoop
#Set path to where hadoop-*-core.jar is available
export HADOOP_MAPRED_HOME=/csh/link/hadoop
#set the path to where bin/hbase is available 配置HBase
export HBASE_HOME=/csh/link/hbase
#Set the path to where bin/hive is available 配置Hive
export HIVE_HOME=/csh/link/hive
#Set the path for where zookeper config dir is 配置Zookeeper
export ZOOCFGDIR=/csh/link/zookeeper/conf
```

### 添加MySQL连接jar包到$SQOOP_HOME/lib目录中 ###

[mysql-connector-java-5.1.31-bin.jar](https://1csh1.github.io/file/Sqoop安装与配置/mysql-connector-java-5.1.31-bin.jar)

### 验证 ###

```
[root@node1 sqoop]# bin/sqoop-version
Warning: /csh/link/sqoop/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /csh/link/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
16/04/10 05:31:47 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
Sqoop 1.4.6
git commit id c0c5a81723759fa575844a0a1eae8f510fa32c25
Compiled by root on Mon Apr 27 14:38:36 CST 2015
```