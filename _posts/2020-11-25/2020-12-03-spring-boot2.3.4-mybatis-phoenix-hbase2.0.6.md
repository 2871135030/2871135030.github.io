---
layout: post
title: SpringBoot2.3.4+Mybatis+Phoenix操作HBase2.0.6
date:   2021-11-12 14:15:08
categories: [linux]
---

#### SpringBoot2.3.4+Mybatis+Phoenix操作HBase2.0.6
##### 1、环境说明
使用单机版安装的hbase，版本为2.0.6，phoenix-core的版本使用5.0.0-HBase-2.0需与hbase版本匹配，使用mybatis直接操作hbase，大部分语法与操作mysql相同，有小部分差异，如`insert or update`在此需改为`upsert`表示插入或更新，同时不支持批量插入语法`insert into table values(),();`

##### 2、配置编写应用代码
* 创建一个用户表，可以在phoenix客户端直接操作

```
create table if not exists ioe_user (
  id bigint not null primary key,
  nick_name varchar(50),
  name varchar(20),
  phone varchar(20),
  address varchar(100),
  gender integer,
  birthday date,
  registry_time date,
  update_time date
);

```
* 编写pom.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.4.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.ioe</groupId>
    <artifactId>phoenix-hbase</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>phoenix-hbase</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
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
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.3.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>


        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.14</version>
        </dependency>

        <dependency>
            <groupId>org.apache.phoenix</groupId>
            <artifactId>phoenix-core</artifactId>
            <version>5.0.0-HBase-2.0</version>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
            </exclusions>
        </dependency>

        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-common</artifactId>
            <version>2.9.2</version>
        </dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>

        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.72</version>
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

* 编写类`User UserMapper Aggregate`

```
package com.ioe.hbase.entity;

import lombok.Getter;
import lombok.Setter;

import java.util.Date;

@Getter
@Setter
public class User {

    private Long id;

    private String nickName;

    private String name;

    private String phone;

    private String address;

    private Integer gender;

    private Date birthday;

    private Date registryTime;

    private Date updateTime;

}

```

```
package com.ioe.hbase.mapper;


import com.ioe.hbase.entity.User;
import com.ioe.hbase.vo.Aggregate;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;
import org.apache.ibatis.annotations.Update;

import java.util.Date;
import java.util.List;

@Mapper
public interface UserMapper {

    @Select("select * from ioe_user order by registry_time desc limit 10 offset 0 ")
    List<User> findLatest();

    @Update("upsert into ioe_user(id,nick_name) values(#{id},#{nickName})")
    void update(@Param("id") Long id, @Param("nickName") String nickName);

    @Update("upsert into ioe_user(id,nick_name,name,phone,address,gender,birthday,registry_time,update_time) " +
            "values(#{id},#{nickName},#{name},#{phone},#{address},#{gender},#{birthday},#{registryTime},#{updateTime})")
    void insert(@Param("id") Long id, @Param("nickName") String nickName, @Param("name") String name, @Param("phone") String phone,
                @Param("address") String address, @Param("gender") Integer gender, @Param("birthday") Date birthday,
                @Param("registryTime") Date registryTime, @Param("updateTime") Date updateTime);

    @Select("select count(*) from ioe_user ")
    Long count();

    @Select("select gender,count(*) count from ioe_user group by gender")
    List<Aggregate> aggregate();

    @Update("delete from ioe_user where id=#{id}")
    void delete(Long id);

}
```

```
package com.ioe.hbase.vo;

import lombok.Getter;
import lombok.Setter;


@Getter
@Setter
public class Aggregate{

    private Integer gender;

    private Long count;

}
```


* 编写配置文件`application.yml`

```
server:
  port: 8081
spring:
  datasource:
    driver-class-name: org.apache.phoenix.jdbc.PhoenixDriver
    url: jdbc:phoenix:192.168.2.5:2181
    type: com.alibaba.druid.pool.DruidDataSource

mybatis:
  type-aliases-package: com.ioe.hbase.entity
  configuration:
    map-underscore-to-camel-case: true

logging:
  level:
    root: info
    com.ioe.hbase: debug

```

* 编写`PhoenixHBaseApplication PhoenixHBaseTest`

```
package com.ioe.hbase;

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@MapperScan("com.ioe.hbase.mapper")
@SpringBootApplication
public class PhoenixHBaseApplication {

    public static void main(String[] args) {
        SpringApplication.run(PhoenixHBaseApplication.class, args);
    }

}

```

```
package com.ioe.hbase;

import com.alibaba.fastjson.JSON;
import com.ioe.hbase.mapper.UserMapper;
import com.ioe.hbase.vo.Aggregate;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.Date;
import java.util.List;

@Slf4j
@SpringBootTest
public class PhoenixHBaseTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    void insert() {
        userMapper.insert(1L, "扬帆起航", "邝小", "13800000000", "北京市东城区", 1, new Date(), new Date(), new Date());
    }

    @Test
    void update() {
        userMapper.update(1L, "扬帆启航");
    }

    @Test
    void aggregate() {
        List<Aggregate> result = userMapper.aggregate();
        result.forEach(e -> log.info(JSON.toJSONString(e)));
    }

    @Test
    void count() {
        log.info("{}", userMapper.count());
    }

    @Test
    void delete() {
        userMapper.delete(1L);
    }

}

```
##### 3、测试运行
运行测试用例，可以看到mybatis对应的sql语句

```
: ==>  Preparing: delete from ioe_user where id=?
: ==> Parameters: 1(Long)
: <==    Updates: 1


: ==>  Preparing: upsert into ioe_user(id,nick_name,name,phone,address,gender,birthday,registry_time,update_time) values(?,?,?,?,?,?,?,?,?)
: ==> Parameters: 1(Long), 扬帆起航(String), 邝小(String), 13800000000(String), 北京市东城区(String), 1(Integer), 2020-01-25 22:47:47.652(Timestamp), 2020-01-25 22:47:47.652(Timestamp), 2020-01-25 22:47:47.652(Timestamp)
: <==    Updates: 1


: ==>  Preparing: upsert into ioe_user(id,nick_name) values(?,?)
: ==> Parameters: 1(Long), 扬帆启航(String)
: <==    Updates: 1


: ==>  Preparing: select count(*) from ioe_user
: ==> Parameters: 
: <==      Total: 1
: 2


: ==>  Preparing: select gender,count(*) count from ioe_user group by gender
: ==> Parameters: 
: <==      Total: 2
: {"count":1,"gender":1}
: {"count":1,"gender":2}
```

##### 4、安装使用DBeaver操作HBase
1）、下载`DBeaver`安装包`dbeaver-ce-7.1.3-win32.win32.x86_64.zip`，解压后进入目录，在配置文件`dbeaver.ini`中增加以下配置指定`jdk`

```
-vm
C:\Program Files\jdk1.8.0_66\bin
```

2）、运行dbeaver.exe，选择创建新连接，选中“Hadoop/BigData”栏的“Apache Phoenix”点击下一步，编辑驱动设置中添加驱动文件`phoenix-5.0.0-HBase-2.0-client.jar`，点选“找到类”即可搜索到对应的驱动类，如图所示；

![image.png](/assets/img/1.png)

![image.png](/assets/img/2.png)

![image.png](/assets/img/3.png)



#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

