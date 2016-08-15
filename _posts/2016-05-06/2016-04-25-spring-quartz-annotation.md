---
layout: post
title: Spring + quartz注解配置使用
date:   2016-04-25 11:08:48
categories: [java,spring,quartz]
---

## Spring + quartz注解配置使用

本文接文<a href="/java/spring/quartz/2016/04/02/spring4-quartz2.html">spring4+quartz2基本配置及应用</a>，使用注解配置

1、编写一个定时器执行类，重要的是使用Scheduled注解

AnnotationQueryJob.java
{% highlight ruby %}
package com.hode.job;

import org.apache.log4j.Logger;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class AnnotationQueryJob {

	private Logger log = Logger.getLogger(getClass());
	
	@Scheduled(cron="0/1 * * * * ?")
	public void execute(){
		log.info("*** ok ***");
	}
	
}
{% endhighlight %}

2、添加spring配置文件applicationContext-annotation.xml，其内容如下
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:util="http://www.springframework.org/schema/util" xmlns:task="http://www.springframework.org/schema/task"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
        http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd">

	<context:component-scan base-package="com" />
    
    <task:annotation-driven/>
    
</beans>
{% endhighlight %}

其中关键标签为<font color="red">&lt;task:annotation-driven/&gt;</font>

当然可根据需要设置executor、scheduler等


3、编写一个测试类完成测试
SchedulerAnnotationTest.java
{% highlight ruby %}
package com.hode;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:*applicationContext-annotation.xml"})
public class SchedulerAnnotationTest {
	
	@Test
	public void test() throws Exception{
		Thread.sleep(300*1000);
	}
	
}

{% endhighlight %}

结束。
