---
title: Spring Boot 配置多个RabbitMQ
date: 2017-01-19 19:53:14
categories: [SpringBoot, RabbitMQ]
tags: [SpringBoot, 技术人生, RabbitMQ]
---

### 闲话 ###
&emsp;&emsp;好久没有写博客了，6月份毕业，因为工作原因，公司上网受限，一直没能把学到的知识点写下来，工作了半年，其实学到的东西也不少，但是现在回忆起来的东西少之又少，有时甚至能在同个问题中踩了几次，越来越觉得及时记录一下学到的东西很重要。
&emsp;&emsp;好了，闲话少说，写下这段时间学习的东西，先记录一下用Spring Boot配置多个RabbitMQ的情况。。。
&emsp;&emsp;最近公司新启动一个新平台的项目，需要用微服务这个这几年很火的概念来做，所以就学习了Spring Boot方面的知识，给同事展示Spring Boot的一些小事例的时候，同事提出了可不可以配置多个RabbitMQ？下面就是在Spring Boot配置多个RabbitMQ的例子。是自己摸索搭建的，也不知道对不对，有其他好的实现方法的网友可以互相交流一下。

### 项目代码构造 ###
![项目代码构造](https://1csh1.github.io/img/Spring Boot 配置多个RabbitMQ/项目代码结构.jpg)

关注点在红框的代码。。。

### 代码 ###
下面就把项目的代码展示下来

#### application.properties ####
配置文件
```
spring.application.name=rabbitmq-hello

# RabbitMQ
spring.rabbitmq.first.host=node9
spring.rabbitmq.first.port=5670
spring.rabbitmq.first.username=guest
spring.rabbitmq.first.password=guest

spring.rabbitmq.second.host=localhost
spring.rabbitmq.second.port=5672
spring.rabbitmq.second.username=guest
spring.rabbitmq.second.password=guest


# MySQL
spring.datasource.url = jdbc:mysql://localhost:3306/cloudtest
spring.datasource.username = root
spring.datasource.password = root
spring.datasource.driverClassName = com.mysql.jdbc.Driver
```

#### HelloApplication.java ####
程序入口

```
package com.paas.springboot.demo01;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloApplication {

    public static void main(String[] args) {
        SpringApplication.run(HelloApplication.class, args);
    }

}
```

#### RabbitConfig.java ####
RabbitMQ配置类

```
package com.paas.springboot.demo01;

import org.springframework.amqp.core.Queue;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.boot.autoconfigure.amqp.SimpleRabbitListenerContainerFactoryConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;

@Configuration
public class RabbitConfig {

	@Bean(name="firstConnectionFactory")
	@Primary
    public ConnectionFactory firstConnectionFactory(
    										@Value("${spring.rabbitmq.first.host}") String host, 
											@Value("${spring.rabbitmq.first.port}") int port,
											@Value("${spring.rabbitmq.first.username}") String username,
											@Value("${spring.rabbitmq.first.password}") String password
											){
		CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
		connectionFactory.setHost(host);
		connectionFactory.setPort(port);
		connectionFactory.setUsername(username);
		connectionFactory.setPassword(password);
		return connectionFactory;
    }
	
	@Bean(name="secondConnectionFactory")
	public ConnectionFactory secondConnectionFactory(
  											@Value("${spring.rabbitmq.second.host}") String host, 
											@Value("${spring.rabbitmq.second.port}") int port,
											@Value("${spring.rabbitmq.second.username}") String username,
											@Value("${spring.rabbitmq.second.password}") String password
											){
		CachingConnectionFactory connectionFactory = new CachingConnectionFactory();
		connectionFactory.setHost(host);
		connectionFactory.setPort(port);
		connectionFactory.setUsername(username);
		connectionFactory.setPassword(password);
		return connectionFactory;
	}
	
	@Bean(name="firstRabbitTemplate")
	@Primary
	public RabbitTemplate firstRabbitTemplate(
											@Qualifier("firstConnectionFactory") ConnectionFactory connectionFactory
											){
		RabbitTemplate firstRabbitTemplate = new RabbitTemplate(connectionFactory);
		return firstRabbitTemplate;
	}
	
	@Bean(name="secondRabbitTemplate")
	public RabbitTemplate secondRabbitTemplate(
											@Qualifier("secondConnectionFactory") ConnectionFactory connectionFactory
											){
		RabbitTemplate secondRabbitTemplate = new RabbitTemplate(connectionFactory);
		return secondRabbitTemplate;
	}
	
	@Bean(name="firstFactory")
	public SimpleRabbitListenerContainerFactory firstFactory(
														SimpleRabbitListenerContainerFactoryConfigurer configurer,
														@Qualifier("firstConnectionFactory") ConnectionFactory connectionFactory		
														) {
		SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		return factory;
	}
	
	@Bean(name="secondFactory")
	public SimpleRabbitListenerContainerFactory secondFactory(
														SimpleRabbitListenerContainerFactoryConfigurer configurer,
														@Qualifier("secondConnectionFactory") ConnectionFactory connectionFactory			
														) {
		SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
		configurer.configure(factory, connectionFactory);
		return factory;
	}
	
    @Bean
    public Queue firstQueue() {
    	System.out.println("configuration firstQueue ........................");
        return new Queue("hello1");
    }
    
    @Bean
    public Object secondQueue() {
    	System.out.println("configuration secondQueue ........................");
    	return new Queue("hello2");
    }
}
```

#### Receiver.java ####
RabbitMQ中的消费者，接收first RabbitMQ中的队列hello1的数据

```
package com.paas.springboot.demo01;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "hello1", containerFactory="firstFactory")
public class Receiver {

    @RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver : " + hello);
    }

}
```

#### Receiver2.java ####
RabbitMQ中的消费者，接收second RabbitMQ中的队列hello2的数据

```
package com.paas.springboot.demo01;

import org.springframework.amqp.rabbit.annotation.RabbitHandler;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
@RabbitListener(queues = "hello2", containerFactory="secondFactory" )
public class Receiver2 {
   
	@RabbitHandler
    public void process(String hello) {
        System.out.println("Receiver : " + hello);
    }

}
```

#### Sender.java ####
RabbitMQ中的生产者，发送消息到first RabbitMQ中的队列hello1和hello2

```
package com.paas.springboot.demo01;

import java.util.Date;
import javax.annotation.Resource;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

@Component
public class Sender {

	@Resource(name="firstRabbitTemplate")
    private RabbitTemplate firstRabbitTemplate;

    public void send1() {
        String context = "hello1 " + new Date();
        System.out.println("Sender : " + context);
        this.firstRabbitTemplate.convertAndSend("hello1", context);
    }
    
    public void send2() {
        String context = "hello2 " + new Date();
        System.out.println("Sender : " + context);
        this.firstRabbitTemplate.convertAndSend("hello2", context);
    }

}
```

#### Sender2.java ####
RabbitMQ中的生产者，发送消息到second RabbitMQ中的队列hello1和hello2

```
package com.paas.springboot.demo01;

import java.util.Date;
import javax.annotation.Resource;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.stereotype.Component;

@Component
public class Sender {

	@Resource(name="firstRabbitTemplate")
    private RabbitTemplate firstRabbitTemplate;

    public void send1() {
        String context = "hello1 " + new Date();
        System.out.println("Sender : " + context);
        this.firstRabbitTemplate.convertAndSend("hello1", context);
    }
    
    public void send2() {
        String context = "hello2 " + new Date();
        System.out.println("Sender : " + context);
        this.firstRabbitTemplate.convertAndSend("hello2", context);
    }

}
```

#### TestDemo01.java ####
测试类，调用Sender发送消息

```
package com.paas.springboot.demo01;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.SpringApplicationConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = HelloApplication.class)
public class TestDemo01 {

    @Autowired
    private Sender sender;
    
    @Autowired
    private Sender2 sender2;

    @Test
    public void hello() throws Exception {
		sender.send1();
		sender.send2();
    }
    
    @Test
    public void hello2() throws Exception {
		sender2.send1();
		sender2.send2();
    }
}
```


#### pom.xml ####
Maven项目中最重要的一个配置文件
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.paas.springboot.demo</groupId>
	<artifactId>springboot</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>springboot Maven Webapp</name>
	<url>http://maven.apache.org</url>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>1.4.3.RELEASE</version>
		<relativePath /> <!-- lookup parent from repository -->
	</parent>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-actuator</artifactId>
		</dependency>
		 <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>com.jayway.jsonpath</groupId>
			<artifactId>json-path</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
		    <groupId>mysql</groupId>
		    <artifactId>mysql-connector-java</artifactId>
		</dependency>
	</dependencies>

	<build>
		<finalName>springboot</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
	
	<repositories>
        <repository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </repository>
    </repositories>
    <pluginRepositories>
        <pluginRepository>
            <id>spring-releases</id>
            <url>https://repo.spring.io/libs-release</url>
        </pluginRepository>
    </pluginRepositories>
    
</project>

```

### 运行&测试 ###
通过运行HelloApplication.java，将程序中的Receiver启动一直监控着队列，然后通过运行TestDemo01.java中的测试案例，发送消息到队列中，这时可以发现运行HelloApplication的程序控制台将刚刚发送的消息打印出来