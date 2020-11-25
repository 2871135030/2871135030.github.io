---
layout: post
title: zuul网关使用ratelimit进行限流控制
date:   2021-11-12 14:15:11
categories: [springcloud]
---


#### zuul网关使用ratelimit进行限流控制

##### 1、依赖版本说明

|依赖|版本|
|-|-|
|spring-boot|1.5.2.RELEASE|
|spring-cloud|Edgware.SR6|
|spring-cloud-zuul-ratelimit|1.7.6.RELEASE|

> 需注意不同版本之间的差异，参数略有不同

##### 2、构建一个maven演示项目gateway-v2
* pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.2.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ioe</groupId>
    <artifactId>gateway-v2</artifactId>
    <version>0.0.1</version>
    <name>gateway-v2</name>
    <description>Gateway project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
        <spring-cloud.version>Edgware.SR6</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>


        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-zuul</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.retry</groupId>
            <artifactId>spring-retry</artifactId>
        </dependency>

        <dependency>
            <groupId>com.marcosbarbero.cloud</groupId>
            <artifactId>spring-cloud-zuul-ratelimit</artifactId>
            <version>1.7.6.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
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
    <repositories>

        <repository>
           <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>

    </repositories>

</project>

```

* GatewayController

```
package com.ioe.gateway.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RequestMapping("/gateway")
@RestController
public class GatewayController {

    @GetMapping("/info")
    public String info() throws Exception {
        //为了方便测试总耗时,休眠一小段时间
        Thread.sleep(800);
        return Thread.currentThread().getName();
    }

}

```
* GatewayApplication

```
package com.ioe.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@EnableZuulProxy
@EnableDiscoveryClient
@SpringBootApplication
public class GatewayApplication {

    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}

```
* application.yaml

```
server:
  port: 80
spring:
  application:
    name: gateway-v2
  redis:
    database: 1
    host: localhost
    password: 2020

zuul:
  routes:
    api-v1:
      path: /v1/**
      url: http://localhost

  ratelimit:
    enabled: true
    repository: REDIS
    policy-list:
      api-v1:
        - limit: 2
          quota: 5
          refresh-interval: 30
          type:
            - url
    default-policy-list:
      - limit: 100
        quota: 5
        refresh-interval: 60
        type:
          - url

hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            timeoutInMilliseconds: 2000

```

* zuul路由说明

> 为了演示方便zuul网关不使用euraka路由到其他服务，直接路由到当前网关的GatewayController，并在接口中增加休眠时间

```
zuul:
  routes:
    api-v1:
      path: /v1/**
      url: http://localhost
```
>  这个配置将演示`http://localhost/v1/gateway/info`转到发`http://localhost/gateway/info`

* zuul ratelimit配置说明

|参数|值|说明|
|-|-|-|
|zuul.ratelimit.enabled|true（默认为false）|是否开启限流功能|
|zuul.ratelimit.repository|可选项`REDIS,CONSUL,JPA`等|使用的存储方式|
|zuul.ratelimit.policy-list|限流参数配置列表|需与routes中配置对应|
|zuul.ratelimit.default-policy-list|默认限流参数配置列表|如果有非默认配置优先匹配非默认规则|


---

|policy-list参数|值|说明|
|-|-|-|
|limit|请求数量限制|如指定60秒内的请求数为10|
|quota|请求时间限制|如指定60秒内接口的请求时间超过5秒即禁止<br>（如上一步10个请求未达到，而接口总时间已超过5秒则限流）|
|refresh-internal|刷新时间周期|如60秒一个时间计算周期|
|type|可选项`ORIGIN,USER,URL`等|使用限流的类型|

> 演示直接使用redis，可以看到请求后redis会有两个维度的请求数据，按上面的配置其中一个key为`gateway-v2:api-v1:/gateway/info`表示当前60秒内的请求总数，另外一个key为`gateway-v2:api-v1:/gateway/info-quota`表示当前60秒内请求的总耗时，计算时间的周期将随redis的失效时间删除。

> 演示application.yml中默认配置(default-policy-list)为60秒限制请求数为100个并且请求的时间总和要小于5秒，若超过100个请求或请求总时间大于5秒则触发限流

##### 3、请求访问演示

> 如api-v1配置中配置了30s只允许2个请求，curl测试如下

```
D:\>curl -L http://localhost/v1/gateway/info
http-nio-80-exec-2
D:\>curl -L http://localhost/v1/gateway/info
http-nio-80-exec-4
D:\>curl -L http://localhost/v1/gateway/info
{"timestamp":1603895041867,"status":429,"error":"Too Many Requests","exception":"com.netflix.zuul.exception.ZuulException","message":"429"}
D:\>
```



#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

