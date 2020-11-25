---
layout: post
title: SpringCloudStream使用kafka
date:   2021-11-12 14:15:00
categories: [java,spring,springcloud]
---

### SpringCloudStream使用kafka
#### 1、安装应用环境
##### 1.1、安装zookeeper
```
1)、下载最新版本安装包apache-zookeeper-3.6.1-bin.tar.gz

2）、解压到安装目录，并将conf目录下文件zoo_sample.cfg拷贝一份命名为zoo.cfg，此文件为zookeeper配置文件

3）、进入安装目录bin中启动zookeeper，
    ./zkServer.sh start
```
##### 1.2、安装kafka
```
1）、下载最新版本安装包kafka_2.13-2.5.0.tgz

2）、解压到安装目录，并完成config/server.properties基本配置

3）、进入安装目录bin中启动kafka
   ./kafka-server-start.sh ../config/server.properties
```
#### 2、spring-cloud中使用kafka完成消息发送与接收
##### 2.1、demo程序代码如下 
* pom.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.1.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.hugesoft</groupId>
    <artifactId>kafka-stream</artifactId>
    <version>0.0.1</version>
    <name>kafka</name>
    <description>Demo project for Spring Boot Kafka</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Hoxton.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-hadoop-hbase</artifactId>
            <version>2.5.0.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.apache.kafka</groupId>
            <artifactId>kafka-streams</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-binder-kafka-streams</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-test-support</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-stream-test-support</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```

* application.yml
```
server:
  port: 8080

spring:
  application:
    name: kafka-stream
  cloud:
    stream:
      kafka:
        binder:
          brokers: 192.168.2.5:9092
      bindings:
        output: # 输出通道
          destination: kafka-stream-topic-test  # 对应的topic
          contentType: text/plain
        input:
          destination: kafka-stream-topic-test
          contentType: text/plain
          group: kafka-stream-test # 指定消费者组


logging:
  level:
    root: info
    org.apache.kafka: warn
```

* Producer.java
```
package com.hugesoft.kafka.mq;

import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.messaging.Source;
import org.springframework.messaging.MessageHeaders;
import org.springframework.messaging.support.MessageBuilder;
import org.springframework.util.MimeTypeUtils;

@Slf4j
@EnableBinding(Source.class)
public class Producer {

    @Autowired
    private Source source;

    public void send(String message) {
        source.output().send(MessageBuilder.withPayload(message)
                .setHeader(MessageHeaders.CONTENT_TYPE, MimeTypeUtils.TEXT_PLAIN_VALUE).build());
    }

}

```
* Consumer.java
```
package com.hugesoft.kafka.mq;

import lombok.extern.slf4j.Slf4j;
import org.springframework.cloud.stream.annotation.EnableBinding;
import org.springframework.cloud.stream.annotation.StreamListener;
import org.springframework.cloud.stream.messaging.Sink;
import org.springframework.kafka.support.KafkaHeaders;
import org.springframework.messaging.Message;
import org.springframework.messaging.handler.annotation.Header;

@Slf4j
@EnableBinding(Sink.class)
public class Consumer {

    @StreamListener(Sink.INPUT)
    public void consume(Message<?> message, @Header(KafkaHeaders.RECEIVED_PARTITION_ID) int partition) {
        log.info("partition [{}], receive [{}]", partition, message);
    }

}

```
##### 2.2、修改kafka分区数量以提升吞吐量
```
进入kafka目录
./kafka-topics.sh --zookeeper localhost:2181 --alter --topic kafka-stream-topic-test  --partitions 10
```

#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

