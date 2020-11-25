---
layout: post
title: springboot2.3.3+hbase-client2.3.1 操作hbase基本使用
date:   2021-11-12 14:15:02
categories: [springcloud]
---

#### springboot2.3.3+hbase-client2.3.1 操作hbase基本使用

由于`spring-data`项目于2019年停止对`hadoop`进行更新，本例中直接使用`hbase-client`原生客户端操作`hbase`，为了方便实验操作，直接使用单机版`hbase`

##### 1、建立一个项目`hbase`演示操作
简单的目录结构如下，`HBaseConfig.java`中配置创建`HBaseService`实例

```
│  pom.xml
│
└─src
    ├─main
    │  ├─java
    │  │  └─com
    │  │      └─ioe
    │  │          └─hbase
    │  │              │  HBaseApplication.java
    │  │              │
    │  │              ├─config
    │  │              │      HBaseConfig.java
    │  │              │
    │  │              └─service
    │  │                      HBaseService.java
    │  │
    │  └─resources
    │      │  application.yml
    │      │
    │      ├─static
    │      └─templates
    └─test
        └─java
            └─com
                └─ioe
                    └─hbase
                            HBaseTest.java
```

具体项目文件内容如下

* pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ioe</groupId>
    <artifactId>hbase</artifactId>
    <version>0.0.1</version>
    <name>hbase</name>
    <description>Hbase2.3 project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.72</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hbase</groupId>
            <artifactId>hbase-client</artifactId>
            <version>2.3.1</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
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
    </dependencies>

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

* HBaseApplication

```
package com.ioe.hbase;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HBaseApplication {

    public static void main(String[] args) {
        SpringApplication.run(HBaseApplication.class, args);
    }

}

```
* HBaseConfig

```
package com.ioe.hbase.config;

import com.ioe.hbase.service.HBaseService;
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class HBaseConfig {

    @Value("${hbase.zookeeper.quorum}")
    private String zookeeper;

    @Bean
    public HBaseService config() {
        org.apache.hadoop.conf.Configuration config = HBaseConfiguration.create();
        config.set("hbase.zookeeper.quorum", zookeeper);
        return new HBaseService(config);
    }

}

```
* HBaseService

```
package com.ioe.hbase.service;

import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.hbase.Cell;
import org.apache.hadoop.hbase.CellUtil;
import org.apache.hadoop.hbase.NamespaceDescriptor;
import org.apache.hadoop.hbase.TableName;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

@Slf4j
public class HBaseService {

    private Connection connection;

    public HBaseService(Configuration config) {
        try {
            connection = ConnectionFactory.createConnection(config);
        } catch (IOException e) {
            log.error("", e);
        }
    }

    public void createNamespace(String namespace) {
        try (Admin admin = connection.getAdmin()) {
            NamespaceDescriptor desc = NamespaceDescriptor.create(namespace).build();
            admin.createNamespace(desc);
            log.info("namespace {} is create success!", namespace);
        } catch (IOException e) {
            log.error("", e);
        }
    }

    public void createTable(String tableName, List<String> columnFamily) {
        try (Admin admin = connection.getAdmin()) {
            List<ColumnFamilyDescriptor> cfDescriptor = columnFamily.stream().map(e -> ColumnFamilyDescriptorBuilder.newBuilder(Bytes.toBytes(e)).build()).collect(Collectors.toList());
            TableDescriptor tableDescriptor = TableDescriptorBuilder.newBuilder(TableName.valueOf(tableName)).setColumnFamilies(cfDescriptor).build();
            if (admin.tableExists(TableName.valueOf(tableName))) {
                log.warn("table {} is exists!", tableName);
            } else {
                admin.createTable(tableDescriptor);
                log.info("table {} is create success!", tableName);
            }
        } catch (IOException e) {
            log.error("", e);
        }
    }

    public void save(String tableName, String rowKey, Map<String, Map<String, String>> data) {
        try {
            Table table = connection.getTable(TableName.valueOf(tableName));
            Put put = new Put(Bytes.toBytes(rowKey));
            data.entrySet().forEach(e -> e.getValue().entrySet().forEach(ee -> {
                put.addColumn(Bytes.toBytes(e.getKey()), Bytes.toBytes(ee.getKey()), Bytes.toBytes(ee.getValue()));
            }));
            table.put(put);
        } catch (Exception e) {
            log.error("", e);
        }
    }

    public Map<String, String> get(String tableName, String rowKey) {
        Map<String, String> map = new HashMap<>();
        try {
            Table table = connection.getTable(TableName.valueOf(tableName));
            Get get = new Get(Bytes.toBytes(rowKey));
            Result result = table.get(get);
            for (Cell c : result.rawCells()) {
                map.put(Bytes.toString(CellUtil.cloneQualifier(c)), Bytes.toString(CellUtil.cloneValue(c)));
                log.info("{}={}={}", Bytes.toString(CellUtil.cloneFamily(c)), Bytes.toString(CellUtil.cloneQualifier(c)), Bytes.toString(CellUtil.cloneValue(c)));
            }
            log.info("{}", JSON.toJSONString(map));
        } catch (Exception e) {
            log.error("", e);
        }
        return map;
    }

}

```
* application.yml

```
logging:
  level:
    root: info

hbase:
  zookeeper:
    quorum: 192.168.2.5:2182

```

* HBaseTest.java

```
package com.ioe.hbase;

import com.alibaba.fastjson.JSON;
import com.ioe.hbase.service.HBaseService;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@Slf4j
@SpringBootTest
class HBaseTest {

    public static final String NAMESPACE = "ioe_ns";

    public static final String TABLE_NAME = "member";

    @Autowired
    private HBaseService service;

    @Test
    void createNamespace() {
        service.createNamespace(NAMESPACE);
    }

    @Test
    void createTable() {
        List<String> cf = new ArrayList<>();
        cf.add("name");
        cf.add("address");
        service.createTable(NAMESPACE + ":" + TABLE_NAME, cf);
    }

    @Test
    void save() {
        Map<String, Map<String, String>> data = new HashMap<>();
        Map<String, String> name = new HashMap<>();
        name.put("chName", "张三");
        name.put("enName", "micheal");
        name.put("nickName", "小张");
        data.put("name", name);
        Map<String, String> address = new HashMap<>();
        address.put("home", "天津");
        address.put("office", "北京");
        data.put("address", address);
        service.save(NAMESPACE + ":" + TABLE_NAME, "zhangsan", data);
    }

    @Test
    void update(){
        Map<String, Map<String, String>> data = new HashMap<>();
        Map<String, String> name = new HashMap<>();
        name.put("nickName", "小张ATest");
        data.put("name", name);
        service.save(NAMESPACE + ":" + TABLE_NAME, "zhangsan", data);
    }

    @Test
    void get() {
        Map<String, String> result = service.get(NAMESPACE + ":" + TABLE_NAME, "zhangsan");
        log.info("查询结果{}", JSON.toJSONString(result));
    }

}

```

##### 2、测试说明

* 可先创建命名空间，再创建表，将表创建到指定命名空间的语法格式为`namespace:tableName`，使用冒号隔开，如不指定命名空间则为默认的`default`命名空间；
* 更新与保存使用的是相同的方法，更新时即覆盖了原有值；


#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

