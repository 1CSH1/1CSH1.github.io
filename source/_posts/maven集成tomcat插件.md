---
title: Maven集成Tomcat插件
date: 2017-01-19 18:53:14
categories: [Maven]
tags: [Maven, 技术人生, Tomcat]
---

### 下载tomcat，在tomcat-users.xml中配置 ###
配置访问tomcat管理页面的用户名和密码

```
<role rolename="manager-gui"/>  
<role rolename="magager-script"/>  
<user username="admin" password="123456" roles="manager-gui,manager-script" />  
```

### 在setting.xml中配置 ###
在maven的配置文件setting.xml中配置访问tomcat的用户名和密码

```
<servers>
<server>
  <id>tomcat.server</id>
  <username>admin</username> 
  <password>123456</password>
</server>
</servers>
```

### 在项目中配置pom.xml ###

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.csh.test</groupId>
	<artifactId>test</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>test Maven Webapp</name>
	<url>http://maven.apache.org</url>
	<build>
		<finalName>test</finalName>
		<sourceDirectory>src</sourceDirectory>
		<plugins>
			<plugin>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.3</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.codehaus.mojo</groupId>
				<artifactId>tomcat-maven-plugin</artifactId>
				<version>1.1</version>
				<configuration>
					<url>http://localhost:8080/manager/text</url>
					<username>admin</username>
					<password>123456</password>
					<path>/</path>
				</configuration>
			</plugin>
		</plugins>
	</build>
	 <dependencies>
	 	<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.36</version>
		</dependency>
		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>javax.servlet-api</artifactId>
			<version>3.0.1</version>
			<scope>provided</scope>
		</dependency>
	 </dependencies>
</project>
```

