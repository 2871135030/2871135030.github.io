---
layout: post
title: 使用maven为不同环境打包，应用不同配置文件
date:   2016-04-01 16:51:39
categories: maven
---

## 使用maven为不同环境打包，应用不同配置文件

使用maven构建工具时，经常需要为不同的环境打不同的war包，如本地环境、开发环境等，各环境的配置不尽相同，下面介绍此功能


1、创建maven项目，首先在src/main/resources分别创建两个环境文件夹local与dev，此两个目录中分别放本地配置与开发环境配置，如log4j.properties，本地为INFO，dev为DEBUG用于区别； 

2、编写pom.xml，指定各环境配置 

{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.qerooy</groupId>
	<artifactId>mvnprofile</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<profiles>
		<profile>
			<id>local</id><!-- 本地环境 -->
			<activation>
				<activeByDefault>true</activeByDefault><!-- 默认使用该环境配置 -->
			</activation>
			<properties>
				<runtime.env>src/main/resources/local</runtime.env>
			</properties>
		</profile>
		
		<profile>
			<id>dev</id><!-- 开发环境 -->
			<properties>
				<runtime.env>src/main/resources/dev</runtime.env>
			</properties>
		</profile>
		
	</profiles>
	
	<build>
		<finalName>profile</finalName>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.1.1</version>
				<configuration>
					<webResources>
						<resource>
							<directory>${runtime.env}</directory>
							<targetPath>WEB-INF/classes</targetPath>
						</resource>
					</webResources>
					<packagingExcludes><!-- 排除多余配置文件 -->
						WEB-INF/classes/local/**,
						WEB-INF/classes/dev/**
					</packagingExcludes>
				</configuration>
			</plugin>
		</plugins>
		
	</build>
	
</project>

{% endhighlight %}


3、如需打本地环境（local）包运行 mvn package便可，打开发环境（dev）包则执行 mvn package -P dev即可。 
