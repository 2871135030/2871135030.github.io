---
layout: post
title: Dubbo全Spring配置方式
date:   2016-04-19 13:03:13
categories: [java,dubbo,spring]
---

## Dubbo的Spring配置方式

DUBBO是一个分布式服务框架，致力于提供高性能和透明化的RPC远程服务调用方案，是阿里巴巴SOA服务化治理方案的核心框架。

1、编写pom.xml，其中包括spring、dubbo基础包，pom.xml内容如下
{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>dubbo</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
		<log4j.version>1.2.17</log4j.version>
	</properties>
	
	<dependencies>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.6.1</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.6.1</version>
		</dependency>
		
		<!--  dubbo start -->
		<dependency> 
		    <groupId>com.alibaba</groupId> 
		    <artifactId>dubbo</artifactId> 
		    <version>2.5.3</version> 
		    <exclusions> 
		        <exclusion> 
		            <groupId>org.springframework</groupId> 
		            <artifactId>spring</artifactId> 
		        </exclusion> 
		    </exclusions> 
		</dependency>
		
		<!--  dubbo end  -->
		
		<!-- zookeeper start --> 
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.5.0-alpha</version>
		</dependency>
		
		<dependency>
		    <groupId>com.101tec</groupId>
		    <artifactId>zkclient</artifactId>
		    <version>0.8</version>
		</dependency>
		<!-- zookeeper end -->
		
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.8</source>
					<target>1.8</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>
</project>
{% endhighlight %}

2、编写一个简单的服务
DemoService.java
{% highlight ruby %}
package com.hode.dubbo.provider;

public interface DemoService {

	public String sayHello(String name);
	
}

{% endhighlight %}
DemoServiceImpl.java
{% highlight ruby %}
package com.hode.dubbo.provider.impl;

import com.hode.dubbo.provider.DemoService;

public class DemoServiceImpl implements DemoService {

	@Override
	public String sayHello(String name) {
		System.out.println("name = "+name);
		return "name is "+name;
	}

}
{% endhighlight %}

3、编写spring配置文件(其中包括了服务提供方和服务消费方)及日志文件，本例为了演示方便不需要启动任何中心节点，只要广播地址一样，就可以互相发现，当然若要使用zookeeper将地址替换即可，各文件内容如下：
applicationContext-provider.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo
        http://code.alibabatech.com/schema/dubbo/dubbo.xsd
        ">
	
	<!-- 提供方应用信息，用于计算依赖关系 -->
	<dubbo:application name="hello-world-app" />
	
	
	<!-- 使用multicast广播注册中心暴露服务地址 -->
	<dubbo:registry address="multicast://224.5.6.7:1234" />
	
	
	<!-- 使用zookeeper注册中心暴露服务地址 
	<dubbo:registry address="zookeeper://192.167.48.128:2181" />
	-->
	
	<!-- 用dubbo协议在20880端口暴露服务 -->
	<dubbo:protocol name="dubbo" port="20880" />
	
	<!-- 声明需要暴露的服务接口 -->
	<dubbo:service interface="com.hode.dubbo.provider.DemoService" ref="demoService" />
	
	<!-- 具体的实现bean -->
	<bean id="demoService" class="com.hode.dubbo.provider.impl.DemoServiceImpl" />
	
	
</beans>
{% endhighlight %}
applicationContext-consumer.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
 
    <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
    <dubbo:application name="consumer-of-helloworld-app"  />
 
    <!-- 使用multicast广播注册中心暴露发现服务地址 -->
    <dubbo:registry address="multicast://224.5.6.7:1234" />
    
    
	<!-- 使用zookeeper注册中心暴露服务地址 
	<dubbo:registry address="zookeeper://192.167.48.128:2181" />-->
 
    <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
    <dubbo:reference id="demoService" interface="com.hode.dubbo.provider.DemoService" />
 
</beans>
{% endhighlight %}
log4j.properties
{% highlight ruby %}
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r %d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n

log4j.logger.com.hode = DEBUG
{% endhighlight %}

4、编写服务提供方代码，并运行，等待服务消费方调用
Provider.java
{% highlight ruby %}
package com.hode;

import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provider {

	public static void main(String[] args) throws Exception{
		
		AbstractApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"applicationContext-provider.xml"});
		context.start();
		System.in.read();
		
	}

}
{% endhighlight %}

5、再编写一个服务消费方，并运行，查看服务调用的情况
Consumer.java
{% highlight ruby %}
package com.hode;

import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.hode.dubbo.provider.DemoService;

public class Consumer {

	public static void main(String[] args) throws Exception{
		AbstractApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"applicationContext-consumer.xml"});
		
		DemoService service = context.getBean(DemoService.class);
		
		String result = service.sayHello("hode");
		
		System.out.println(String.format("consumer result[%s]",result));
	}

}


{% endhighlight %}

结束。
