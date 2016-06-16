---
layout: post
title: spring4+quartz2基本配置及应用
date:   2016-04-01 22:51:39
categories: java
---

## spring4+quartz2基本配置及应用

1、新建maven工程，添加spring4.0及quartz2.2依赖，pom.xml文件内容如下 

{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.qerooy</groupId>
	<artifactId>spring-quartz</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
		<log4j.version>1.2.17</log4j.version>
		<quartz.version>2.2.1</quartz.version>
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
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>

		<dependency>
			<groupId>org.quartz-scheduler</groupId>
			<artifactId>quartz</artifactId>
			<version>${quartz.version}</version>
		</dependency>
		
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>3.2.3.RELEASE</version>
			<scope>test</scope>
		</dependency>

	</dependencies>

	<build />
</project>

{% endhighlight %}

2、新建一个简单Job（本例演示继承QuartzJobBean），其简单实现如下 

{% highlight ruby %}
public class QueryJob extends QuartzJobBean{
	
	Logger log = Logger.getLogger(getClass());
	
	public void query(){
		log.info(" log "+new Date());
	}

	@Override
	protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
		query();
	}
}

{% endhighlight %}

3、增加spring配置，同时增加log4j日志 
spring配置文件applicationContext.xml如下 
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd">


	<bean id="jobDetail" class="org.springframework.scheduling.quartz.JobDetailFactoryBean">
		<property name="jobClass" value="com.qerooy.job.QueryJob"></property>
		<property name="durability" value="true" />
		
	</bean>
	
	<bean id="cronTriggerBean" class="org.springframework.scheduling.quartz.CronTriggerFactoryBean">
        <property name="jobDetail" ref="jobDetail"></property>
        <property name="cronExpression" value="* * * * * ?"></property>
    </bean>

    <bean id="trigger" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
        <property name="triggers">
            <list>
                <ref bean="cronTriggerBean"/>
            </list>
        </property>
    </bean>  
    

</beans>

{% endhighlight %}
log4j简单配置日志log4j.properties文件如下 
{% highlight ruby %}
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-5r [%-17t] %-5p %c %x - %m%n

{% endhighlight %}

4、最后编写一个junit测试类，观察job运行情况 
{% highlight ruby %}
package com.qerooy;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:*applicationContext.xml"})
public class SchedulerTest {
	
	@Test
	public void test() throws Exception{
		Thread.sleep(300*1000);
	}
	
}
{% endhighlight %}
