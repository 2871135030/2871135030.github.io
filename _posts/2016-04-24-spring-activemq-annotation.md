---
layout: post
title: Spring + Activemq注解配置使用
date:   2016-04-24 12:12:48
categories: [java,spring]
---

## Spring + Activemq注解配置使用

本文接文<a href="/java/spring/2016/04/17/spring-activemq.html">Spring + Activemq基本使用</a>，将消息接收配置简化，使用注解配置

1、首先升级spring所依赖包版本，将上文中pom.xml的spring版本改为4.3.1.RELEASE，旧版本中无@JmsListener注解，其余依赖不变

2、编写一个消息监听类，此类使用了注解JmsListener，并指定了队列名

ReceiveMessageAnnotationListener.java
{% highlight ruby %}
package com.hode.listener;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.ObjectMessage;
import javax.jms.TextMessage;

import org.apache.log4j.Logger;
import org.springframework.jms.annotation.JmsListener;
import org.springframework.stereotype.Component;

import com.hode.model.UserMessage;

@Component
public class ReceiveMessageAnnotationListener {

	private static final Logger log = Logger.getLogger(ReceiveMessageAnnotationListener.class);

	/**
	 * 处理消息
	 * @param message
	 */
	@JmsListener(destination="UserQueue")
	public void onMessageHandle(Message message) {
		if (message instanceof TextMessage) {
			TextMessage text = (TextMessage) message;
			try {
				log.info("Received text message:" + text.getText());
			} catch (JMSException e) {
				e.printStackTrace();
			}
		}else if(message instanceof ObjectMessage){
			ObjectMessage om = (ObjectMessage)message;
			UserMessage um;
			try {
				um = (UserMessage)om.getObject();
				log.info("Received Object message:" + um);
			} catch (JMSException e) {
				e.printStackTrace();
			}
		}else{
			log.warn("nothing proper message");
		}
	}
	
}
{% endhighlight %}

3、添加spring配置文件applicationContext-mq-annotation.xml，其内容如下
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:jms="http://www.springframework.org/schema/jms"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/jms http://www.springframework.org/schema/jms/spring-jms-4.1.xsd ">
	
	<context:component-scan base-package="com" />
	
	<jms:annotation-driven/>
	
	<bean id="jmsListenerContainerFactory" class="org.springframework.jms.config.DefaultJmsListenerContainerFactory" >
		<property name="connectionFactory">
			<bean class="org.apache.activemq.ActiveMQConnectionFactory">
				<property name="brokerURL">
					<value>tcp://192.167.48.128:61616</value>
				</property>
			</bean>
		</property>
	</bean>
	
</beans>
{% endhighlight %}
其中关键标签为<font color="red">&lt;jms:annotation-driven/&gt;</font>

<font color="red">另id必须命名为jmsListenerContainerFactory</font> 


4、编写一个主类监听消息
AppAnnotationServer.java
{% highlight ruby %}
package com.hode;

import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 服务
 */
public class AppAnnotationServer {

	@SuppressWarnings("resource")
	public static void main(String[] args) throws Exception{
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext-mq-annotation.xml");
		context.registerShutdownHook();
	}

}
{% endhighlight %}

同样使用上文中发送消息用例，往mq队列发送消息完成测试。

结束。
