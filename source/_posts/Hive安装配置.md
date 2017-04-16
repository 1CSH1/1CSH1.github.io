---
title: Hive安装配置
date: 2016-04-06 19:22:38
categories: [Hive]
tags: [Hive, 技术人生]
---
1.下载解压安装包
下载链接：[apache-hive-1.2.1-bin.tar.gz](https://1csh1.github.io/file/Hive安装配置/apache-hive-1.2.1-bin.tar.gz)

```
tar -xvf apache-hive-1.2.1-bin.tar.gz
```

2.启动Hadoop
可以参考：[Hadoop2.x配置HA](https://1csh1.github.io/2016/03/27/Hadoop2.x%E9%85%8D%E7%BD%AEHA/)

4.在$HIVE_HOME/bin目录下执行

```
./hive
```

5.创建表test

```
[root@node1 bin]# ./hive
Logging initialized using configuration in jar:file:/csh/software/apache-hive-1.2.1-bin/lib/hive-common-1.2.1.jar!/hive-log4j.properties
hive> create table test (a int);
OK
Time taken: 41.704 seconds
```

6.访问HDFS文件系统可以查看到test文件夹【创建一个表，就会在/user/hive/warehouse文件下创建对应的文件夹】
![Hive安装配置](https://1csh1.github.io/img/Hive安装配置/Hive安装配置.png)
