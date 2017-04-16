---
title: Sqoop HDFS导入到MySQL
date: 2016-04-21 16:28:01
categories: [Sqoop]
tags: [Sqoop, 技术人生]
---
### 在MySQL中创建表 ###

```
CREATE TABLE `tree1` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `treeNumber` varchar(100) NOT NULL,
  `productinformationId` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) DEFAULT CHARSET=utf8;
```

### 执行命令 ###

```
sqoop export --connect jdbc:mysql://node1:3306/phx \--username root \--table tree1 -m 1 \--export-dir /sqoop/tree2
```

### 结果 ###

```
[root@node1 conf]# sqoop export --connect jdbc:mysql://node1:3306/phx \--username root \--table tree1 -m 1 \--export-dir /sqoop/tree2
Warning: /csh/link/sqoop/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /csh/link/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
16/04/19 01:10:23 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
16/04/19 01:10:23 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
16/04/19 01:10:23 INFO tool.CodeGenTool: Beginning code generation
16/04/19 01:10:24 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tree1` AS t LIMIT 1
16/04/19 01:10:24 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tree1` AS t LIMIT 1
16/04/19 01:10:24 INFO orm.CompilationManager: HADOOP_MAPRED_HOME is /csh/link/hadoop
Note: /tmp/sqoop-root/compile/8efdea8893437a1359c77de6d1bd2395/tree1.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
16/04/19 01:10:28 INFO orm.CompilationManager: Writing jar file: /tmp/sqoop-root/compile/8efdea8893437a1359c77de6d1bd2395/tree1.jar
16/04/19 01:10:29 INFO mapreduce.ExportJobBase: Beginning export of tree1
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/csh/software/hadoop-2.7.2/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/csh/software/hbase-1.1.4/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
16/04/19 01:10:30 INFO Configuration.deprecation: mapred.jar is deprecated. Instead, use mapreduce.job.jar
16/04/19 01:10:38 INFO Configuration.deprecation: mapred.reduce.tasks.speculative.execution is deprecated. Instead, use mapreduce.reduce.speculative
16/04/19 01:10:38 INFO Configuration.deprecation: mapred.map.tasks.speculative.execution is deprecated. Instead, use mapreduce.map.speculative
16/04/19 01:10:38 INFO Configuration.deprecation: mapred.map.tasks is deprecated. Instead, use mapreduce.job.maps
16/04/19 01:10:38 INFO client.RMProxy: Connecting to ResourceManager at node1/192.168.161.11:8032
16/04/19 01:10:48 INFO input.FileInputFormat: Total input paths to process : 1
16/04/19 01:10:48 INFO input.FileInputFormat: Total input paths to process : 1
16/04/19 01:10:49 INFO mapreduce.JobSubmitter: number of splits:1
16/04/19 01:10:49 INFO Configuration.deprecation: mapred.map.tasks.speculative.execution is deprecated. Instead, use mapreduce.map.speculative
16/04/19 01:10:49 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1461026006811_0003
16/04/19 01:10:51 INFO impl.YarnClientImpl: Submitted application application_1461026006811_0003
16/04/19 01:10:51 INFO mapreduce.Job: The url to track the job: http://node1:8088/proxy/application_1461026006811_0003/
16/04/19 01:10:51 INFO mapreduce.Job: Running job: job_1461026006811_0003
16/04/19 01:11:33 INFO mapreduce.Job: Job job_1461026006811_0003 running in uber mode : false
16/04/19 01:11:33 INFO mapreduce.Job:  map 0% reduce 0%
16/04/19 01:12:24 INFO mapreduce.Job:  map 100% reduce 0%
16/04/19 01:12:27 INFO mapreduce.Job: Job job_1461026006811_0003 completed successfully
16/04/19 01:12:27 INFO mapreduce.Job: Counters: 30
	File System Counters
		FILE: Number of bytes read=0
		FILE: Number of bytes written=137080
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=251
		HDFS: Number of bytes written=0
		HDFS: Number of read operations=4
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=0
	Job Counters 
		Launched map tasks=1
		Rack-local map tasks=1
		Total time spent by all maps in occupied slots (ms)=45106
		Total time spent by all reduces in occupied slots (ms)=0
		Total time spent by all map tasks (ms)=45106
		Total vcore-milliseconds taken by all map tasks=45106
		Total megabyte-milliseconds taken by all map tasks=46188544
	Map-Reduce Framework
		Map input records=7
		Map output records=7
		Input split bytes=122
		Spilled Records=0
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=258
		CPU time spent (ms)=1680
		Physical memory (bytes) snapshot=86806528
		Virtual memory (bytes) snapshot=2086785024
		Total committed heap usage (bytes)=17235968
	File Input Format Counters 
		Bytes Read=0
	File Output Format Counters 
		Bytes Written=0
16/04/19 01:12:27 INFO mapreduce.ExportJobBase: Transferred 251 bytes in 109.7035 seconds (2.288 bytes/sec)
16/04/19 01:12:27 INFO mapreduce.ExportJobBase: Exported 7 records.
```

### 在MySQL中查看tree1表中数据 ###
![数据库表](https://1csh1.github.io/img/Sqoop%20HDFS导入到MySQL/数据库表.jpg)



