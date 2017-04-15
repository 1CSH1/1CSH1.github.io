---
title: 配置Hadoop2.x的HDFS、MapReduce来运行WordCount程序
date: 2016-03-25 12:53:14
categories: [Hadoop]
tags: [Hadoop2.x, 技术人生, WordCount]
---
| 主机  |  HDFS | MapReduce |
|:-----:|:-------:|:-------------:|
|node1 | NameNode | ResourceManager|
|node2 |SecondaryNameNode & DataNode | NodeManager |
|node3 | DataNode | NodeManager |
|node4 | DataNode | NodeManager |

1.配置hadoop-env.sh

```
export JAVA_HOME=/csh/link/jdk
```

2.配置core-site.xml

```
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://node1:9000</value>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/csh/hadoop/hadoop2.7.2/tmp</value>
</property>
```

3.配置hdfs-site.xml

```
<property>
    <name>dfs.namenode.http-address</name>
    <value>node1:50070</value>
</property>
<property>
    <name>dfs.namenode.secondary.http-address</name>
    <value>node2:50090</value>
</property>
<property>
    <name>dfs.namenode.name.dir</name>
    <value>/csh/hadoop/hadoop2.7.2/name</value>
</property>
<property>
    <name>dfs.datanode.data.dir</name>
    <value>/csh/hadoop/hadoop2.7.2/data</value>
</property>
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
```

4.配置mapred-site.xml

```
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

5.配置yarn-site.xml

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

6.配置masters

```
node2
```

7.配置slaves

```
node2
node3
node4
```

8.启动Hadoop

```
bin/hadoop namenode -format
sbin/start-dfs.sh
sbin/start-yarn.sh

```

9.运行WordCount程序

```
//创建文件wc.txt
echo "I love Java I love Hadoop I love BigData Good Good Study, Day Day Up" > wc.txt
//创建HDFS中的文件
hdfs dfs -mkdir -p /input/wordcount/
//将wc.txt上传到HDFS中
hdfs dfs -put wc.txt /input/wordcount
//运行WordCount程序
hadoop jar /csh/software/hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /input/wordcount/ /output/wordcount/
```

10.结果

```
[root@node1 sbin]# hadoop jar /csh/software/hadoop-2.7.2/share/hadoop/mapreduce/hadoapreduce-examples-2.7.2.jar wordcount /input/wordcount/ /output/wordcount/
16/03/24 19:26:48 INFO client.RMProxy: Connecting to ResourceManager at node1/192.161.11:8032
16/03/24 19:26:56 INFO input.FileInputFormat: Total input paths to process : 1
16/03/24 19:26:56 INFO mapreduce.JobSubmitter: number of splits:1
16/03/24 19:26:57 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_145887175_0001
16/03/24 19:26:59 INFO impl.YarnClientImpl: Submitted application application_145887175_0001
16/03/24 19:27:00 INFO mapreduce.Job: The url to track the job: http://node1:8088/prapplication_1458872237175_0001/
16/03/24 19:27:00 INFO mapreduce.Job: Running job: job_1458872237175_0001
16/03/24 19:28:13 INFO mapreduce.Job: Job job_1458872237175_0001 running in uber modfalse
16/03/24 19:28:13 INFO mapreduce.Job:  map 0% reduce 0%
16/03/24 19:30:07 INFO mapreduce.Job:  map 100% reduce 0%
16/03/24 19:31:13 INFO mapreduce.Job:  map 100% reduce 33%
16/03/24 19:31:16 INFO mapreduce.Job:  map 100% reduce 100%
16/03/24 19:31:23 INFO mapreduce.Job: Job job_1458872237175_0001 completed successfu
16/03/24 19:31:24 INFO mapreduce.Job: Counters: 49
	File System Counters
		FILE: Number of bytes read=106
		FILE: Number of bytes written=235387
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=174
		HDFS: Number of bytes written=64
		HDFS: Number of read operations=6
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=2
	Job Counters 
		Launched map tasks=1
		Launched reduce tasks=1
		Data-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=116501
		Total time spent by all reduces in occupied slots (ms)=53945
		Total time spent by all map tasks (ms)=116501
		Total time spent by all reduce tasks (ms)=53945
		Total vcore-milliseconds taken by all map tasks=116501
		Total vcore-milliseconds taken by all reduce tasks=53945
		Total megabyte-milliseconds taken by all map tasks=119297024
		Total megabyte-milliseconds taken by all reduce tasks=55239680
	Map-Reduce Framework
		Map input records=4
		Map output records=15
		Map output bytes=129
		Map output materialized bytes=106
		Input split bytes=105
		Combine input records=15
		Combine output records=9
		Reduce input groups=9
		Reduce shuffle bytes=106
		Reduce input records=9
		Reduce output records=9
		Spilled Records=18
		Shuffled Maps =1
		Failed Shuffles=0
		Merged Map outputs=1
		GC time elapsed (ms)=1468
		CPU time spent (ms)=6780
		Physical memory (bytes) snapshot=230531072
		Virtual memory (bytes) snapshot=4152713216
		Total committed heap usage (bytes)=134795264
	Shuffle Errors
		BAD_ID=0
		CONNECTION=0
		IO_ERROR=0
		WRONG_LENGTH=0
		WRONG_MAP=0
		WRONG_REDUCE=0
	File Input Format Counters 
		Bytes Read=69
	File Output Format Counters 
		Bytes Written=64
[root@node1 sbin]# hdfs dfs -cat /output/wordcount/*
BigData	1
Day	2
Good	2
Hadoop	1
I	3
Java	1
Study,	1
Up	1
love	3
```

