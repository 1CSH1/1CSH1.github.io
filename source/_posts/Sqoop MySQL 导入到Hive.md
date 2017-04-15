---
title: Sqoop MySQL 导入到Hive
date: 2016-04-21 15:52:20
categories: [Sqoop]
tags: [Sqoop, 技术人生]
---
### 将数据库phx中的tree表的数据导入到Hive中 ###
命令：

```
sqoop import --connect jdbc:mysql://node1:3306/phx \--username root \--table tree \--hive-import \--hive-overwrite \--create-hive-table \--hive-table tree1 \--target-dir /sqoop/tree3
```

结果：

```
[root@node1 conf]# sqoop import --connect jdbc:mysql://node1:3306/phx \--username root \--table tree \--hive-import \--hive-overwrite \--create-hive-table \--hive-table tree1 \--target-dir /sqoop/tree3
Warning: /csh/link/sqoop/../hcatalog does not exist! HCatalog jobs will fail.
Please set $HCAT_HOME to the root of your HCatalog installation.
Warning: /csh/link/sqoop/../accumulo does not exist! Accumulo imports will fail.
Please set $ACCUMULO_HOME to the root of your Accumulo installation.
16/04/18 21:20:00 INFO sqoop.Sqoop: Running Sqoop version: 1.4.6
16/04/18 21:20:00 INFO tool.BaseSqoopTool: Using Hive-specific delimiters for output. You can override
16/04/18 21:20:00 INFO tool.BaseSqoopTool: delimiters with --fields-terminated-by, etc.
16/04/18 21:20:00 INFO manager.MySQLManager: Preparing to use a MySQL streaming resultset.
16/04/18 21:20:00 INFO tool.CodeGenTool: Beginning code generation
16/04/18 21:20:00 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tree` AS t LIMIT 1
16/04/18 21:20:00 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tree` AS t LIMIT 1
16/04/18 21:20:00 INFO orm.CompilationManager: HADOOP_MAPRED_HOME is /csh/link/hadoop
Note: /tmp/sqoop-root/compile/9cff004a8ff405b712c978864c2775df/tree.java uses or overrides a deprecated API.
Note: Recompile with -Xlint:deprecation for details.
16/04/18 21:20:03 INFO orm.CompilationManager: Writing jar file: /tmp/sqoop-root/compile/9cff004a8ff405b712c978864c2775df/tree.jar
16/04/18 21:20:03 WARN manager.MySQLManager: It looks like you are importing from mysql.
16/04/18 21:20:03 WARN manager.MySQLManager: This transfer can be faster! Use the --direct
16/04/18 21:20:03 WARN manager.MySQLManager: option to exercise a MySQL-specific fast path.
16/04/18 21:20:03 INFO manager.MySQLManager: Setting zero DATETIME behavior to convertToNull (mysql)
16/04/18 21:20:03 INFO mapreduce.ImportJobBase: Beginning import of tree
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/csh/software/hadoop-2.7.2/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/csh/software/hbase-1.1.4/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
16/04/18 21:20:04 INFO Configuration.deprecation: mapred.jar is deprecated. Instead, use mapreduce.job.jar
16/04/18 21:20:09 INFO Configuration.deprecation: mapred.map.tasks is deprecated. Instead, use mapreduce.job.maps
16/04/18 21:20:10 INFO client.RMProxy: Connecting to ResourceManager at node1/192.168.161.11:8032
16/04/18 21:20:23 INFO db.DBInputFormat: Using read commited transaction isolation
16/04/18 21:20:23 INFO db.DataDrivenDBInputFormat: BoundingValsQuery: SELECT MIN(`id`), MAX(`id`) FROM `tree`
16/04/18 21:20:24 INFO mapreduce.JobSubmitter: number of splits:4
16/04/18 21:20:25 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1461026006811_0001
16/04/18 21:20:27 INFO impl.YarnClientImpl: Submitted application application_1461026006811_0001
16/04/18 21:20:28 INFO mapreduce.Job: The url to track the job: http://node1:8088/proxy/application_1461026006811_0001/
16/04/18 21:20:28 INFO mapreduce.Job: Running job: job_1461026006811_0001
16/04/18 21:21:01 INFO mapreduce.Job: Job job_1461026006811_0001 running in uber mode : false
16/04/18 21:21:01 INFO mapreduce.Job:  map 0% reduce 0%
16/04/18 21:22:00 INFO mapreduce.Job:  map 25% reduce 0%
16/04/18 21:23:13 INFO mapreduce.Job:  map 50% reduce 0%
16/04/18 21:23:14 INFO mapreduce.Job:  map 100% reduce 0%
16/04/18 21:23:21 INFO mapreduce.Job: Job job_1461026006811_0001 completed successfully
16/04/18 21:23:22 INFO mapreduce.Job: Counters: 31
	File System Counters
		FILE: Number of bytes read=0
		FILE: Number of bytes written=549568
		FILE: Number of read operations=0
		FILE: Number of large read operations=0
		FILE: Number of write operations=0
		HDFS: Number of bytes read=398
		HDFS: Number of bytes written=373
		HDFS: Number of read operations=16
		HDFS: Number of large read operations=0
		HDFS: Number of write operations=8
	Job Counters 
		Killed map tasks=4
		Launched map tasks=7
		Other local map tasks=7
		Total time spent by all maps in occupied slots (ms)=585524
		Total time spent by all reduces in occupied slots (ms)=0
		Total time spent by all map tasks (ms)=585524
		Total vcore-milliseconds taken by all map tasks=585524
		Total megabyte-milliseconds taken by all map tasks=599576576
	Map-Reduce Framework
		Map input records=21
		Map output records=21
		Input split bytes=398
		Spilled Records=0
		Failed Shuffles=0
		Merged Map outputs=0
		GC time elapsed (ms)=722
		CPU time spent (ms)=15670
		Physical memory (bytes) snapshot=394702848
		Virtual memory (bytes) snapshot=8350515200
		Total committed heap usage (bytes)=68792320
	File Input Format Counters 
		Bytes Read=0
	File Output Format Counters 
		Bytes Written=373
16/04/18 21:23:22 INFO mapreduce.ImportJobBase: Transferred 373 bytes in 193.0206 seconds (1.9324 bytes/sec)
16/04/18 21:23:22 INFO mapreduce.ImportJobBase: Retrieved 21 records.
16/04/18 21:23:23 INFO manager.SqlManager: Executing SQL statement: SELECT t.* FROM `tree` AS t LIMIT 1
16/04/18 21:23:23 INFO hive.HiveImport: Loading uploaded data into Hive
16/04/18 21:25:54 INFO hive.HiveImport: 
16/04/18 21:25:54 INFO hive.HiveImport: Logging initialized using configuration in file:/csh/software/apache-hive-1.2.1-bin/conf/hive-log4j.properties
16/04/18 21:25:57 INFO hive.HiveImport: SLF4J: Class path contains multiple SLF4J bindings.
16/04/18 21:25:57 INFO hive.HiveImport: SLF4J: Found binding in [jar:file:/csh/software/hadoop-2.7.2/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
16/04/18 21:25:57 INFO hive.HiveImport: SLF4J: Found binding in [jar:file:/csh/software/hbase-1.1.4/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
16/04/18 21:25:57 INFO hive.HiveImport: SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
16/04/18 21:25:57 INFO hive.HiveImport: SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
16/04/18 21:27:27 INFO hive.HiveImport: OK
16/04/18 21:27:28 INFO hive.HiveImport: Time taken: 23.5 seconds
16/04/18 21:27:30 INFO hive.HiveImport: Loading data to table default.tree1
16/04/18 21:27:35 INFO hive.HiveImport: Table default.tree1 stats: [numFiles=4, numRows=0, totalSize=373, rawDataSize=0]
16/04/18 21:27:35 INFO hive.HiveImport: OK
16/04/18 21:27:35 INFO hive.HiveImport: Time taken: 8.27 seconds
16/04/18 21:27:36 INFO hive.HiveImport: Hive import complete.
16/04/18 21:27:40 INFO hive.HiveImport: Export directory is contains the _SUCCESS file only, removing the directory.
```

浏览HDFS如下图：
![HDFS](https://1csh1.github.io/img/Sqoop%20MySQL%20导入到Hive/HDFS.jpg)

在Hive中检查是否有数据：

```
hive> select * from tree1;
OK
1	466464684640	1
2	466464684641	2
3	466464684642	3
4	466464684643	1
5	466464684644	2
6	466464684645	3
7	466464684646	1
8	466464684647	2
9	466464684648	3
10	466464684649	1
11	4664646846410	2
12	4664646846411	3
13	4664646846412	1
14	4664646846413	2
15	4664646846414	3
16	4664646846415	1
17	4664646846416	2
18	4664646846417	3
19	4664646846418	1
20	4664646846419	2
21	111111	1
Time taken: 2.622 seconds, Fetched: 21 row(s)
```

### 参数说明 ###

 | 参数 | 	说明 | 
 |:---|:---| 
 | --hive-home <dir> | 	Hive的安装目录，可以通过该参数覆盖掉默认的hive目录 | 
 | --hive-overwrite | 	覆盖掉在hive表中已经存在的数据 | 
 | --create-hive-table | 	默认是false，如果目标表已经存在了，那么创建任务会失败 | 
 | --hive-table | 	后面接要创建的hive表 | 
 | --table | 	指定关系数据库表名 | 
