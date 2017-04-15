---
title: Flume集群搭建
date: 2016-04-21 18:20:43
categories: [Flume]
tags: [Flume, 技术人生]
---
### 概念 ###
集群的意思是多台机器，最少有2台机器，一台机器从数据源中获取数据，将数据传送到另一台机器上，然后输出。接下来就要实现Flume集群搭建。文中的集群如下图所示。
![架构](https://1csh1.github.io/img/Flume集群搭建/架构.jpg)
这里我们需要2台机器，node1作为push推送数据，node2作为pull获取数据后显示出来。

### 配置pull.conf ###
【在node2机器上操作】
在conf目录下创建pull.conf文件
```
touch pull.conf
```
编辑pull.conf
```
#汇总数据代理的配置文件pull.conf
#Name the components on this agent
a1.sources= r1
a1.sinks= k1
a1.channels= c1
 
#Describe/configure the source
a1.sources.r1.type= avro
a1.sources.r1.channels= c1
a1.sources.r1.bind= node2
a1.sources.r1.port= 44444
 
#Describe the sink
a1.sinks.k1.type= logger
a1.sinks.k1.channel = c1
 
#Use a channel which buffers events in memory
a1.channels.c1.type= memory
a1.channels.c1.keep-alive= 10
a1.channels.c1.capacity= 100000
a1.channels.c1.transactionCapacity= 100000
```

### 配置push.conf ###
【在node1机器上操作】
在conf目录下创建push.conf文件
```
touch push.conf
```
编辑push.conf
```
#推数据代理的配置文件push.conf
#Name the components on this agent
a2.sources= r1
a2.sinks= k1
a2.channels= c1
 
#Describe/configure the source
a2.sources.r1.type= spooldir
a2.sources.r1.spoolDir= /csh/hadoop/flume/logs
a2.sources.r1.channels= c1
 
#Use a channel which buffers events in memory
a2.channels.c1.type= memory
a2.channels.c1.keep-alive= 10
a2.channels.c1.capacity= 100000
a2.channels.c1.transactionCapacity= 100000
 
#Describe/configure the source
a2.sinks.k1.type= avro
a2.sinks.k1.channel= c1
a2.sinks.k1.hostname= node2
a2.sinks.k1.port= 44444
```

### 创建spoolDir目录 ###
【在node1中进行该操作】
根据push.conf中的配置 a2.sources.r1.spoolDir参数，创建目录，如果不先创建目录，则启动时会报错
```
mkdir -p /csh/hadoop/flume/logs
```

### 启动作为pull的主机 ###
【本文为node2主机】
```
[root@node2 flume]# flume-ng agent -c conf -f conf/pull.conf -n a1 -Dflume.root.logger=INFO,console
```

显示如下信息则为启动成功

```
2016-04-20 00:08:15,550 (conf-file-poller-0) [INFO - org.apache.flume.node.PollingPropertiesFileConfigurationProvider$FileWatcherRunnable.run(PollingPropertiesFileConfigurationProvider.java:133)] Reloading configuration file:conf/pull.conf
2016-04-20 00:08:15,573 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:931)] Added sinks: k1 Agent: a1
2016-04-20 00:08:15,573 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1017)] Processing:k1
2016-04-20 00:08:15,574 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration$AgentConfiguration.addProperty(FlumeConfiguration.java:1017)] Processing:k1
2016-04-20 00:08:15,621 (conf-file-poller-0) [INFO - org.apache.flume.conf.FlumeConfiguration.validateConfiguration(FlumeConfiguration.java:141)] Post-validation flume configuration contains configuration for agents: [a1]
2016-04-20 00:08:15,622 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:145)] Creating channels
2016-04-20 00:08:15,658 (conf-file-poller-0) [INFO - org.apache.flume.channel.DefaultChannelFactory.create(DefaultChannelFactory.java:42)] Creating instance of channel c1 type memory
2016-04-20 00:08:15,672 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.loadChannels(AbstractConfigurationProvider.java:200)] Created channel c1
2016-04-20 00:08:15,677 (conf-file-poller-0) [INFO - org.apache.flume.source.DefaultSourceFactory.create(DefaultSourceFactory.java:41)] Creating instance of source r1, type avro
2016-04-20 00:08:15,732 (conf-file-poller-0) [INFO - org.apache.flume.sink.DefaultSinkFactory.create(DefaultSinkFactory.java:42)] Creating instance of sink: k1, type: logger
2016-04-20 00:08:15,735 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.getConfiguration(AbstractConfigurationProvider.java:114)] Channel c1 connected to [r1, k1]
2016-04-20 00:08:15,750 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:138)] Starting new configuration:{ sourceRunners:{r1=EventDrivenSourceRunner: { source:Avro source r1: { bindAddress: node2, port: 44444 } }} sinkRunners:{k1=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@ea5ba80 counterGroup:{ name:null counters:{} } }} channels:{c1=org.apache.flume.channel.MemoryChannel{name: c1}} }
2016-04-20 00:08:15,782 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:145)] Starting Channel c1
2016-04-20 00:08:15,784 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:160)] Waiting for channel: c1 to start. Sleeping for 500 ms
2016-04-20 00:08:15,897 (lifecycleSupervisor-1-2) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:120)] Monitored counter group for type: CHANNEL, name: c1: Successfully registered new MBean.
2016-04-20 00:08:15,901 (lifecycleSupervisor-1-2) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:96)] Component type: CHANNEL, name: c1 started
2016-04-20 00:08:16,285 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:173)] Starting Sink k1
2016-04-20 00:08:16,288 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:184)] Starting Source r1
2016-04-20 00:08:16,298 (lifecycleSupervisor-1-2) [INFO - org.apache.flume.source.AvroSource.start(AvroSource.java:228)] Starting Avro source r1: { bindAddress: node2, port: 44444 }...
2016-04-20 00:08:16,951 (lifecycleSupervisor-1-2) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:120)] Monitored counter group for type: SOURCE, name: r1: Successfully registered new MBean.
2016-04-20 00:08:16,952 (lifecycleSupervisor-1-2) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:96)] Component type: SOURCE, name: r1 started
2016-04-20 00:08:16,959 (lifecycleSupervisor-1-2) [INFO - org.apache.flume.source.AvroSource.start(AvroSource.java:253)] Avro source r1 started.
```

### 启动作为push的主机 ###
【本文为node1主机】
```
[root@node1 flume]# flume-ng agent -n a2 -c conf -f conf/push.conf -Dflume.root.logger=INFO,console
```
显示如下信息则为启动成功
```
2016-04-20 00:11:58,196 (conf-file-poller-0) [INFO - org.apache.flume.node.AbstractConfigurationProvider.getConfiguration(AbstractConfigurationProvider.java:114)] Channel c1 connected to [r1, k1]
2016-04-20 00:11:58,226 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:138)] Starting new configuration:{ sourceRunners:{r1=EventDrivenSourceRunner: { source:Spool Directory source r1: { spoolDir: /csh/hadoop/flume/logs } }} sinkRunners:{k1=SinkRunner: { policy:org.apache.flume.sink.DefaultSinkProcessor@6b089e25 counterGroup:{ name:null counters:{} } }} channels:{c1=org.apache.flume.channel.MemoryChannel{name: c1}} }
2016-04-20 00:11:58,236 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:145)] Starting Channel c1
2016-04-20 00:11:58,360 (lifecycleSupervisor-1-1) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:120)] Monitored counter group for type: CHANNEL, name: c1: Successfully registered new MBean.
2016-04-20 00:11:58,361 (lifecycleSupervisor-1-1) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:96)] Component type: CHANNEL, name: c1 started
2016-04-20 00:11:58,362 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:173)] Starting Sink k1
2016-04-20 00:11:58,369 (conf-file-poller-0) [INFO - org.apache.flume.node.Application.startAllComponents(Application.java:184)] Starting Source r1
2016-04-20 00:11:58,372 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.source.SpoolDirectorySource.start(SpoolDirectorySource.java:78)] SpoolDirectorySource source starting with directory: /csh/hadoop/flume/logs
2016-04-20 00:11:58,388 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.sink.AbstractRpcSink.start(AbstractRpcSink.java:289)] Starting RpcSink k1 { host: node2, port: 44444 }...
2016-04-20 00:11:58,409 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:120)] Monitored counter group for type: SINK, name: k1: Successfully registered new MBean.
2016-04-20 00:11:58,409 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:96)] Component type: SINK, name: k1 started
2016-04-20 00:11:58,409 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.sink.AbstractRpcSink.createConnection(AbstractRpcSink.java:206)] Rpc sink k1: Building RpcClient with hostname: node2, port: 44444
2016-04-20 00:11:58,410 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.sink.AvroSink.initializeRpcClient(AvroSink.java:126)] Attempting to create Avro Rpc client.
2016-04-20 00:11:58,458 (lifecycleSupervisor-1-0) [WARN - org.apache.flume.api.NettyAvroRpcClient.configure(NettyAvroRpcClient.java:634)] Using default maxIOWorkers
2016-04-20 00:11:58,536 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.register(MonitoredCounterGroup.java:120)] Monitored counter group for type: SOURCE, name: r1: Successfully registered new MBean.
2016-04-20 00:11:58,536 (lifecycleSupervisor-1-4) [INFO - org.apache.flume.instrumentation.MonitoredCounterGroup.start(MonitoredCounterGroup.java:96)] Component type: SOURCE, name: r1 started
2016-04-20 00:11:59,263 (lifecycleSupervisor-1-0) [INFO - org.apache.flume.sink.AbstractRpcSink.start(AbstractRpcSink.java:303)] Rpc sink k1 started.
```
这时pull主机【本文为node2】输出信息表示连接成功
```
2016-04-20 00:11:58,875 (New I/O server boss #1 ([id: 0x71ba9ce2, /192.168.161.12:44444])) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x7d9299a9, /192.168.161.11:44003 => /192.168.161.12:44444] OPEN
2016-04-20 00:11:58,880 (New I/O  worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x7d9299a9, /192.168.161.11:44003 => /192.168.161.12:44444] BOUND: /192.168.161.12:44444
2016-04-20 00:11:58,884 (New I/O  worker #1) [INFO - org.apache.avro.ipc.NettyServer$NettyServerAvroHandler.handleUpstream(NettyServer.java:171)] [id: 0x7d9299a9, /192.168.161.11:44003 => /192.168.161.12:44444] CONNECTED: /192.168.161.11:44003
```

### 测试 ###
在push主机中【本文为node1】的spoolDir目录【本文为/csh/hadoop/flume/logs】中创建test.log
```
vi test.log
# 输入内容 hello flume
```
这时push主机【本文为node1】中命令行输出如下
```
2016-04-20 00:13:09,274 (pool-4-thread-1) [INFO - org.apache.flume.client.avro.ReliableSpoolingFileEventReader.readEvents(ReliableSpoolingFileEventReader.java:258)] Last read took us just up to a file boundary. Rolling to the next file, if there is one.
2016-04-20 00:13:09,275 (pool-4-thread-1) [INFO - org.apache.flume.client.avro.ReliableSpoolingFileEventReader.rollCurrentFile(ReliableSpoolingFileEventReader.java:348)] Preparing to move file /csh/hadoop/flume/logs/test.log to /csh/hadoop/flume/logs/test.log.COMPLETED
```

pull主机【本文为node2】中命令行输出如下
```
2016-04-20 00:13:21,344 (SinkRunner-PollingRunner-DefaultSinkProcessor) [INFO - org.apache.flume.sink.LoggerSink.process(LoggerSink.java:94)] Event: { headers:{} body: 68 65 6C 6C 6F 20 66 6C 75 6D 65                hello flume }
```

证明Flume集群搭建成功

我们可以发现test.log被改名为test.log.COMPLETED