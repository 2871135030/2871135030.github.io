---
layout: post
title: Spring+Dubbo annotation配置
date:   2016-04-22 13:23:46
categories: [java,spring,dubbo]
---

## Spring+Dubbo annotation配置

本文继<a href="/java/dubbo/2016/04/19/spring-dubbo.html">dubbo spring配置使用</a>，上文中使用的xml，dubbo提供了注解方式使代码更加简洁

1、编写pom.xml，其中包括spring、dubbo基础包，pom.xml内容请参考上文（在此省略）

2、编写一个简单的服务
FooService.java
{% highlight ruby %}
package com.hode.dubbo.provider;

public interface FooService {

	public String sayHello(String name);
	
}

{% endhighlight %}
FooServiceImpl.java，注意需使用alibaba的Service注解
{% highlight ruby %}
package com.hode.dubbo.provider.impl;

import com.alibaba.dubbo.config.annotation.Service;
import com.hode.dubbo.provider.FooService;

@Service(version="1.0.0")
public class FooServiceImpl implements FooService {

	@Override
	public String sayHello(String name) {
		System.out.println("name = "+name);
		return "name is "+name;
	}

}

{% endhighlight %}
FooServiceClient.java，此为客户端调用类，注意需使用alibaba的Service注解
{% highlight ruby %}
package com.hode.dubbo.consumer;

import org.springframework.stereotype.Component;

import com.alibaba.dubbo.config.annotation.Reference;
import com.hode.dubbo.provider.FooService;

@Component
public class FooServiceClient {

	@Reference(version="1.0.0")
	private FooService fooService;

	public FooService getFooService() {
		return fooService;
	}

	public void setFooService(FooService fooService) {
		this.fooService = fooService;
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
	
	<!-- 公共信息，也可以用dubbo.properties配置 -->
	<dubbo:application name="annotation-provider" />
	
	<dubbo:registry address="multicast://224.5.6.7:1234" />
	
	
	<!-- 扫描注解包路径，多个包用逗号分隔，不填package表示扫描ApplicationContext中所有的类 -->
	<dubbo:annotation package="com.hode.dubbo.provider" />
		
	
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
 
	
	<dubbo:application name="annotation-consumer" />
	
	<dubbo:registry address="multicast://224.5.6.7:1234" />
	
	<dubbo:annotation package="com.hode.dubbo.consumer" />
	 
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
package com.hode.dubbo;

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
package com.hode.dubbo;

import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.hode.dubbo.consumer.FooServiceClient;
import com.hode.dubbo.provider.FooService;

public class Consumer {

	public static void main(String[] args) throws Exception{
		AbstractApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"applicationContext-consumer.xml"});
		
		FooServiceClient client = context.getBean(FooServiceClient.class);
		
		FooService service = client.getFooService();
		
		String result = service.sayHello("hode");
		
		System.out.println(String.format("consumer result[%s]",result));
	}

}


{% endhighlight %}


结束。
