---
layout: post
title: Spring + RocketMQ使用介绍
date:   2016-05-22 14:33:58
categories: [spring]
---

## Spring + RocketMQ使用介绍

本文所介绍环境为win7环境下运行，
从官方github中（https://github.com/alibaba/RocketMQ）下载RocketMQ-master.zip，版本为v3.5.8，解压并进入根目录，运行命令install.bat，
安装完成后进入目录\target\alibaba-rocketmq-broker\alibaba-rocketmq\bin，打开两个命令行窗口，分别使用以下命令启动rocketmq

```markdown
启动nameserver
mqnamesrv.exe
启动broker
mqbroker -n 127.0.0.1:9876
```

1、编写pom.xml，

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>rocketmq</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	
	<properties>
		<spring.version>4.3.2.RELEASE</spring.version>
		<junit.version>4.12</junit.version>
		<log4j.version>1.2.17</log4j.version>
		<rocketmq.version>3.2.6</rocketmq.version>
		<slf4j.version>1.7.12</slf4j.version>
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
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>
		
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		
		<dependency>
            <groupId>com.alibaba.rocketmq</groupId>
            <artifactId>rocketmq-client</artifactId>
            <version>${rocketmq.version}</version>
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
```

2、编写spring配置文件applicationContext-consumer.xml,applicationContext-producer.xml以及log4j.properties，内容如下

applicationContext-consumer.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd">

	<bean id="producer" class="com.hode.rocketmq.Consumer" init-method="init" destroy-method="destroy">
		<constructor-arg name="consumerGroup" value="rocketmq-test" />
		<constructor-arg name="namesrvAddr" value="127.0.0.1:9876" />
		<constructor-arg name="instanceName" value="test" />
		<constructor-arg name="topic" value="testTopic" />
		<constructor-arg name="messageListener" ref="messageListener" />
	</bean>
	
	<bean id="messageListener" class="com.hode.rocketmq.StringMessageListener" />
	
</beans>

```

applicationContext-producer.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd">

	<bean id="producer" class="com.hode.rocketmq.Producer" init-method="init" destroy-method="destroy">
		<constructor-arg name="producerGroup" value="rocketmq-test" />
		<constructor-arg name="namesrvAddr" value="127.0.0.1:9876" />
		<constructor-arg name="instanceName" value="test" />
	</bean>
	
</beans>
```

log4j.properties

```markdown
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r %d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n

log4j.logger.com.hode=DEBUG

```

3、编写类StringMessageListener.java,Producer.java,Consumer.java

StringMessageListener.java

```markdown
package com.hode.rocketmq;

import java.util.List;

import org.apache.log4j.Logger;

import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.common.message.MessageExt;

public class StringMessageListener implements MessageListenerConcurrently{

	private Logger log = Logger.getLogger(getClass());
	
	@Override
	public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs,ConsumeConcurrentlyContext context) {
		for (MessageExt msg : msgs) {
            log.info("msg : " + new String(msg.getBody()));
        }
		return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
	}

}

```

Producer.java

```markdown
package com.hode.rocketmq;

import org.apache.log4j.Logger;

import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;

/**
 * 生产
 */
public class Producer {
	
	protected Logger log = Logger.getLogger(getClass());
	
	private String producerGroup;
	
	private String namesrvAddr;
	
	private String instanceName;
	
	private DefaultMQProducer producer;
	
	public DefaultMQProducer getProducer() {
		return producer;
	}

	public Producer(String producerGroup,String namesrvAddr,String instanceName){
		this.producerGroup = producerGroup;
		this.namesrvAddr = namesrvAddr;
		this.instanceName = instanceName;
	}
	
	public void init() throws MQClientException{
		log.info("start init DefaultMQProducer...");
		producer = new DefaultMQProducer(producerGroup);
		producer.setNamesrvAddr(namesrvAddr);
		producer.setInstanceName(instanceName);
		producer.start();
		log.info("DefaultMQProducer init success.");
	}
	
	public void destroy(){
		log.info("start destroy DefaultMQProducer...");
		producer.shutdown();
		log.info("DefaultMQProducer destroy success.");
	}

}

```

Consumer.java

```markdown
package com.hode.rocketmq;

import org.apache.log4j.Logger;

import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;

public class Consumer {

	private Logger log = Logger.getLogger(getClass());
	
	private DefaultMQPushConsumer consumer;
	
	private String consumerGroup;
	
	private String namesrvAddr;
	
	private String instanceName;
	
	private String topic;
	
	private MessageListenerConcurrently messageListener;
	
	public Consumer(String consumerGroup,String namesrvAddr,String instanceName,String topic,MessageListenerConcurrently messageListener){
		this.consumerGroup = consumerGroup;
		this.namesrvAddr = namesrvAddr;
		this.instanceName = instanceName;
		this.topic = topic;
		this.messageListener = messageListener;
	}
	
	public void init() throws Exception{
		log.info("start init DefaultMQPushConsumer...");
		consumer = new DefaultMQPushConsumer(consumerGroup);
		consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET); //从队列头部开始消费
		consumer.setNamesrvAddr(namesrvAddr);
		consumer.setInstanceName(instanceName);
		consumer.subscribe(topic, "*");
		consumer.registerMessageListener(messageListener);
		consumer.start();
		log.info("DefaultMQPushConsumer init ok.");
	}

	public void destroy(){
		log.info("start destroy DefaultMQPushConsumer...");
		consumer.shutdown();
		log.info("DefaultMQPushConsumer destroy success.");
	}
	
	public DefaultMQPushConsumer getConsumer() {
		return consumer;
	}

}

```

4、编写测试类

ProducerTest.java

```markdown
package com.hode;

import java.util.Date;

import org.apache.log4j.Logger;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.SendCallback;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;
import com.hode.rocketmq.Producer;

public class ProducerTest {

	private static Logger log = Logger.getLogger(ProducerTest.class);
	
	private static ApplicationContext context;
	
	public static void main(String[] args) throws Exception{
		context = new ClassPathXmlApplicationContext("classpath:applicationContext-producer.xml");
		Producer producer = context.getBean(Producer.class);
		DefaultMQProducer p = producer.getProducer();
		
		String message = "test messgae"+new Date();
		Message msg = new Message("testTopic",message.getBytes());
		log.info(message);
		p.send(msg, new SendCallback(){

			@Override
			public void onSuccess(SendResult sendResult) {
				log.info(sendResult.getSendStatus().name());
				log.info("onSuccess");
				
				producer.destroy();
			}

			@Override
			public void onException(Throwable e) {
				log.error("onException");
			}
		});
		
	}
	
}

```

ConsumerTest.java

```markdown
package com.hode;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.hode.rocketmq.Consumer;

public class ConsumerTest {

	private static ApplicationContext context;
		
	public static void main(String[] args) throws Exception{
		context = new ClassPathXmlApplicationContext("classpath:applicationContext-consumer.xml");
		Consumer consumer = context.getBean(Consumer.class);
		
		Thread.sleep(20*1000);
		
		System.out.println("end");
		consumer.destroy();
	}
	
}

```

分别运行生产端及消费端完成测试，结束。

<h4><a href="/rar/rocketmq.rar"><b>Demo代码下载</b></a></h4>
