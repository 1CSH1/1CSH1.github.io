---
title: Eurkea 简单介绍
date: 2017-04-15 10:00:00
categories: [Eureka]
tags: [Eureka]
---

个人博客原文：[]()

> 摘要：本文简单介绍 Eureka

​# Eureka 是什么？

Eureka 是基于 REST 的一种用于负载均衡和故障切换的一个中间服务。是 Netflix 公司开源的一个产品，在微服务架构中用于服务注册与发现，也被 Spring Cloud 集成。

# 架构

图中每一个 Eureka Server 都是一个集群，每个集群可以互相备份。Application Service 是一个服务应用，而 Application Client 是一个客户端应用，两者的区别在于 Application Service 是被调用方，而 Application Client 是调用方，两者都注册到 Eureka Server 上，并在 Eureka Client 里面一直和 Eureka Server 通讯，拉取注册在 Eureka Server 上的程序信息到本地。
![Eureka架构图](https://1csh1.github.io/img/eureka-introduce/eureka-introduce01.png)

# 配置 Eureka Client

## 配置 pom.xml

```xml
<dependency>
  <groupId>com.netflix.eureka</groupId>
  <artifactId>eureka-client</artifactId>
  <version>1.1.16</version>
 </dependency>
```

## 配置文件

默认的配置文件为 eureka-client.properties 。默认的配置文件例子可以在这个链接看[conf](https://github.com/Netflix/eureka/tree/master/eureka-examples/conf)

最少的配置参数是下面几个

```
Application Name (eureka.name)
Application Port (eureka.port)
Virtual HostName (eureka.vipAddress)
Eureka Service Urls (eureka.serviceUrls)
```
如果要看最全的配置，可以查看源代码[EurekaInstanceConfig.java](https://github.com/Netflix/eureka/blob/master/eureka-client/src/main/java/com/netflix/appinfo/EurekaInstanceConfig.java)，[EurekaClientConfig.java](https://github.com/Netflix/eureka/blob/master/eureka-client/src/main/java/com/netflix/discovery/EurekaClientConfig.java)

# 配置 Eureka Server

## 下载 eureka-server-xxx.war

[eureka-server](http://search.maven.org/#search%7Cga%7C1%7Ceureka-server)

## 配置文件

默认的配置文件为 eureka-server.properties

## 运行 eureka server
将 eureka-server-xxx.war 放在 tomcat 的 webapps 目录下，并修改名字为 eureka.war



