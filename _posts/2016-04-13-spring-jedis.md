---
layout: post
title: Spring4+jedis基本操作
date:   2016-04-13 15:32:57
categories: [java,spring,redis]
---

## Spring4+jedis基本操作

首先安装好redis,并启动

1、编写pom.xml文件，添加jedis、spring等配置，pom文件如下

{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>spring-jedis</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
	</properties>
	
	<dependencies>
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-redis</artifactId>
			<version>1.6.1.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-beans</artifactId>
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
		    <groupId>redis.clients</groupId>
		    <artifactId>jedis</artifactId>
		    <version>2.8.0</version>
		    <type>jar</type>
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
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.8.2</version>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
			<scope>test</scope>
		</dependency>
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

2、编写applicationContext.xml文件、log4j.properties文件，分别如下
applicationContext.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">
		<property name="maxIdle" value="100" />
		<property name="maxTotal" value="300" />
		<property name="maxWaitMillis" value="3000" />
	</bean>
	
	<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="hostName" value="192.167.48.128" />
		<property name="port" value="6379" />
		<property name="password" value="redis2016" />
		<property name="poolConfig" ref="poolConfig"/>
	</bean>
	
	<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate" >
		<property name="connectionFactory" ref="connectionFactory" />
		<property name="keySerializer">
			<bean class="org.springframework.data.redis.serializer.StringRedisSerializer" />
		</property>
	</bean>
	
</beans>
{% endhighlight %}
log4j.properties
{% highlight ruby %}
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n

log4j.logger.com.hode = DEBUG
{% endhighlight %}

3、编写一个测试用例，完成redis的基本操作，代码如下（详见代码注释）：

{% highlight ruby %}
package com.hode;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;
import java.util.Date;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class JedisTest {

	@Autowired
	RedisTemplate<String,Object> template; //此处RedisTemplate的key,value分别定义为String,Object
	
	@Test
	public void test(){
		
		//设置值
		template.opsForValue().set("name", "hode-test");
		
		//取值
		System.out.println(template.opsForValue().get("name"));
		
		//设置对象(对象需序列化)
		template.opsForValue().set("user", new User("hode-test",12,new Date()));
		
		//获取对象
		User u = (User)template.opsForValue().get("user");
		System.out.println(u);
	}
	
}

class User implements Serializable{
	private static final long serialVersionUID = -3692788205421627317L;
	private String name;
	private int age;
	private Date birthday;
	public User(String name, int age, Date birthday) {
		this.name = name;
		this.age = age;
		this.birthday = birthday;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	public Date getBirthday() {
		return birthday;
	}
	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}
	@Override
	public String toString() {
		return "User [name=" + name + ", age=" + age + ", birthday=" + birthday
				+ "]";
	}
}

{% endhighlight %}

结束。
