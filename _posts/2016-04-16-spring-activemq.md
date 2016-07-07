---
layout: post
title: Spring + Activemq基本使用
date:   2016-04-16 19:11:57
categories: java
---

## Spring + Activemq基本使用

1、安装，Apache activemq的安装比较简单，下载压缩包apache-activemq-5.12.1-bin.tar.gz到目录/software
{% highlight ruby %}
cd /software
tar -xzvf apache-activemq-5.12.1-bin.tar.gz
cd /software/apache-activemq-5.12.1/bin
./activemq start
{% endhighlight %}

启动完成后，进入控制台 http://192.167.48.128:8161/admin/  会提示录入用户名及密码，用户名及密码均为admin，即可查看队列信息


2、编写一个消息发送端，在此演示发送一个字符串及一个对象，对象需序列化。
编写pom.xml
{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>mqsend</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
		<log4j.version>1.2.17</log4j.version>
		<junit.version>4.11</junit.version>
		<commons-lang.version>2.5</commons-lang.version>
		<commons-beanutils.version>1.9.2</commons-beanutils.version>
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
			<artifactId>spring-jms</artifactId>
			<version>${spring.version}</version>
		</dependency>
	
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>

		<dependency>
			<groupId>commons-beanutils</groupId>
			<artifactId>commons-beanutils</artifactId>
			<version>${commons-beanutils.version}</version>
		</dependency>
		
		<dependency>
			<groupId>commons-lang</groupId>
			<artifactId>commons-lang</artifactId>
			<version>${commons-lang.version}</version>
			<type>jar</type>
			<scope>compile</scope>
		</dependency>
		
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.12</version>
		</dependency>
		
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>${junit.version}</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
			<scope>test</scope>
		</dependency>
		
		<dependency>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-core</artifactId>
			<version>5.7.0</version>
		</dependency>
		
		<dependency>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-pool</artifactId>
			<version>5.7.0</version>
		</dependency>
		
	</dependencies>
	
	<build>
		
		<pluginManagement>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			
		</plugins>
		</pluginManagement>
	</build>
	
</project>
{% endhighlight %}

编写一个对象，用于模拟mq发送接收
{% highlight ruby %}
package com.hode.model;

import java.io.Serializable;

public class Message implements Serializable{

	private static final long serialVersionUID = -5454767231530853722L;

}

package com.hode.model;

import java.util.Date;

public class UserMessage extends Message {

	private static final long serialVersionUID = 5614702318469487173L;

	
	private int id;
	
	private String name;
	
	private double salary;
	
	private Date birthday;
	
	private int age;

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public double getSalary() {
		return salary;
	}

	public void setSalary(double salary) {
		this.salary = salary;
	}

	public Date getBirthday() {
		return birthday;
	}

	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	@Override
	public String toString() {
		return "UserMessage [id=" + id + ", name=" + name + ", salary="
				+ salary + ", birthday=" + birthday + ", age=" + age + "]";
	}
}

{% endhighlight %}
	
编写一个消息发送类
{% highlight ruby %}
package com.hode;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.ObjectMessage;
import javax.jms.Session;
import javax.jms.TextMessage;

import org.apache.log4j.Logger;
import org.springframework.jms.core.JmsTemplate;
import org.springframework.jms.core.MessageCreator;

public class MessageSender {
	private static final Logger log = Logger.getLogger(MessageSender.class);
	private JmsTemplate jmsTemplate;

	public void setJmsTemplate(JmsTemplate jmsTemplate) {
		this.jmsTemplate = jmsTemplate;
	}

	public void sendMessage(final String message) {
		log.info("Send message: " + message);
		jmsTemplate.send(new MessageCreator() {
			public Message createMessage(Session session) throws JMSException {
				TextMessage textMessage = session.createTextMessage(message);
				return textMessage;
			}

		});
	}
	
	public void sendObjectMessage(final com.hode.model.Message message) {
		jmsTemplate.send(new MessageCreator() {
			public Message createMessage(Session session) throws JMSException {
				ObjectMessage objectMessage = session.createObjectMessage(message);
				return objectMessage;
			}

		});
	}
}

{% endhighlight %}

编写spring配置文件applicationContext-mq.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:context="http://www.springframework.org/schema/context" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd ">
	
	<context:component-scan base-package="com" />
	
	<bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory" destroy-method="stop">
		<property name="connectionFactory">
			<bean class="org.apache.activemq.ActiveMQConnectionFactory">
				<property name="brokerURL">
					<value>tcp://192.167.48.128:61616?jms.redeliveryPolicy.maximumRedeliveries=3</value>
				</property>
			</bean>
		</property>
		<property name="maxConnections" value="100"></property>
	</bean>
	
	<bean id="messageQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg index="0" value="UserQueue" />
    </bean>
    
	<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">
		<property name="connectionFactory" ref="connectionFactory"></property>
		<property name="defaultDestination" ref="messageQueue"></property>
	</bean>
	
	<bean id="messageSender" class="com.hode.MessageSender">
		<property name="jmsTemplate" ref="jmsTemplate"></property>
	</bean>
	
</beans>
{% endhighlight %}
编写log4j.properties文件
{% highlight ruby %}
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r [%t] %d{yyyyMMdd HHmmss,SSS} %-5p %c %x - %m%n

log4j.logger.com.hode=DEBUG
{% endhighlight %}

最后编写一个测试用例
{% highlight ruby %}
package com.hode;

import java.util.Date;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.hode.model.UserMessage;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:applicationContext-mq.xml"})
public class SenderTest {

	@Autowired
	MessageSender messageSender;
	
	@Test
	public void test() {
		messageSender.sendMessage("test"); //发送字符串
		
		UserMessage user = new UserMessage();
		user.setAge(1);
		user.setBirthday(new Date());
		user.setId(3);
		user.setName("hode");
		user.setSalary(30000);	
		messageSender.sendObjectMessage(user); //发送对象(对象需序列化)
	}

}

{% endhighlight %}

运行发送测试用例，将发送一些消息到mq队列

3、编写消息接收程序(其中部分文件与发送程序一样)
编写pom.xml
{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>mqreceive</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<!-- 与发送端一样 -->
</project>
{% endhighlight %}
编写接收监听程序(其中使用的Message,UserMessage与发送端一样，此处省略)
{% highlight ruby %}
package com.hode.listener;

import javax.jms.JMSException;
import javax.jms.Message;
import javax.jms.MessageListener;
import javax.jms.ObjectMessage;
import javax.jms.TextMessage;

import org.apache.log4j.Logger;

import com.hode.model.UserMessage;

public class ReceiveMessageListener implements  MessageListener{

	private static final Logger log = Logger.getLogger(ReceiveMessageListener.class);

	@Override
	public void onMessage(Message message) {
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
编写一个服务启动程序
{% highlight ruby %}
package com.hode;

import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * 服务
 */
public class AppServer {

	@SuppressWarnings("resource")
	public static void main(String[] args) throws Exception{
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:applicationContext-mq.xml");
		context.registerShutdownHook();
	}

}
{% endhighlight %}

编写spring配置文件applicationContext-mq.xml(log4j.properties文件与发送端一样)
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd ">
	
	<context:component-scan base-package="com" />
	
	<bean id="listenerContainer"
		class="org.springframework.jms.listener.DefaultMessageListenerContainer">
		<property name="connectionFactory" ref="connectionFactory"></property>
		<property name="destination" ref="messageQueue"></property>
		<property name="messageListener" ref="receiveMessageListener"></property>
		<property name="sessionTransacted" value="true"/>
	</bean>
	
	<bean id="connectionFactory" class="org.apache.activemq.pool.PooledConnectionFactory"
		destroy-method="stop">
		<property name="connectionFactory">
			<bean class="org.apache.activemq.ActiveMQConnectionFactory">
				<property name="brokerURL">
					<value>tcp://192.167.48.128:61616</value>
				</property>
			</bean>
		</property>
		<property name="maxConnections" value="100"></property>
	</bean>
	
	<bean id="messageQueue" class="org.apache.activemq.command.ActiveMQQueue">
        <constructor-arg index="0" value="UserQueue" />
    </bean>
    
	<bean id="receiveMessageListener" class="com.hode.listener.ReceiveMessageListener"></bean>
	
</beans>
{% endhighlight %}

运行AppServer主程序后将监听mq队列里的消息，可执行发送端测试用例查看消息接收情况。


结束。
