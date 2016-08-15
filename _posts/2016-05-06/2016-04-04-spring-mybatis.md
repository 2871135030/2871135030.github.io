---
layout: post
title: spring+mybaits配置
date:   2016-04-04 13:11:36
categories: [java,spring]
---

## spring+mybaits配置

本文介绍spring+mybatis配置，数据库使用mysql，mybatis代码使用mybatis-generator生成 

1、创建maven项目，加入spring、mybatis、mysql、proxool依赖，并加入mybatis-generator插件，pom.xml如下
{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.qerooy</groupId>
	<artifactId>spring-mybatis</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	
	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
		<mybatis.version>3.2.2</mybatis.version>
		<mybatis-spring.version>1.2.0</mybatis-spring.version>
		<log4j.version>1.2.17</log4j.version>
		<mysql.version>5.1.29</mysql.version>
		<proxool.version>0.9.1</proxool.version>
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
			<artifactId>spring-tx</artifactId>
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
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>${mybatis.version}</version>
		</dependency>

		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>${mybatis-spring.version}</version>
		</dependency>
		
		<dependency>
			<groupId>mysql</groupId>
		    <artifactId>mysql-connector-java</artifactId>
		    <version>${mysql.version}</version>
		</dependency>
		
		<dependency>
			<groupId>proxool</groupId>
			<artifactId>proxool</artifactId>
			<version>${proxool.version}</version>
		</dependency>
		
		<dependency>
			<groupId>com.cloudhopper.proxool</groupId>
			<artifactId>proxool-cglib</artifactId>
			<version>${proxool.version}</version>
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
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.mybatis.generator</groupId>
				<artifactId>mybatis-generator-maven-plugin</artifactId>
				<version>1.3.2</version>  
			</plugin>
		</plugins>
	</build>
</project>
{% endhighlight %}

2、创建数据库qerooy，并创建一个user表； 
{% highlight ruby %}
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(30) NOT NULL,
  `password` varchar(50) NOT NULL,
  `remark` varchar(50) DEFAULT NULL,
  `create_time` datetime NOT NULL,
  `update_time` datetime NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

{% endhighlight %}
3、编写mybatis-generator代码生成插件配置文件generatorConfig.xml 
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
  PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
  "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
  <classPathEntry location="D:\.m2\repository\mysql\mysql-connector-java\5.1.29\mysql-connector-java-5.1.29.jar" />

  <context id="Mysql2Tables" targetRuntime="MyBatis3">
  
  	<plugin type="org.mybatis.generator.plugins.RowBoundsPlugin">
  	</plugin>
  	 
	<commentGenerator>
		<property name="suppressAllComments" value="false" />
		<property name="suppressDate" value="true" />
	</commentGenerator>
    
    <jdbcConnection driverClass="com.mysql.jdbc.Driver"
        connectionURL="jdbc:mysql://localhost:3306/qerooy"
        userId="root"
        password="123jkl">
    </jdbcConnection>

    <javaTypeResolver >
      <property name="forceBigDecimals" value="false" />
    </javaTypeResolver>

    <javaModelGenerator targetPackage="com.qerooy.model" targetProject="src/main/java">
      <property name="enableSubPackages" value="false" />
      <property name="trimStrings" value="true" />
    </javaModelGenerator>

    <sqlMapGenerator targetPackage="map"  targetProject="src/main/resources/">
      <property name="enableSubPackages" value="false" />
    </sqlMapGenerator>

    <javaClientGenerator type="XMLMAPPER" targetPackage="com.qerooy.mapper"  targetProject="src/main/java">
      <property name="enableSubPackages" value="false" />
    </javaClientGenerator>
    
    <table schema="root" tableName="user" domainObjectName="User" >
    	<generatedKey column="id" sqlStatement="MYSQL" identity="true" />
    </table>
 
  </context>
</generatorConfiguration>
{% endhighlight %}

执行命令 mvn -Dmybatis.generator.overwrite=true mybatis-generator:generate生成mybatis代码 

4、编写mybatis分页插件 
{% highlight ruby %}
package com.qerooy.base.dialect;

public class Dialect {

	public boolean supportsLimit() {
		return false;
	}

	public boolean supportsLimitOffset() {
		return supportsLimit();
	}

	public String getLimitString(String sql, int offset, int limit) {
		return getLimitString(sql, offset, Integer.toString(offset), limit, Integer.toString(limit));
	}

	public String getLimitString(String sql, int offset, String offsetPlaceholder, int limit, String limitPlaceholder) {
		throw new UnsupportedOperationException("paged queries not supported");
	}

}
{% endhighlight %}

{% highlight ruby %}
package com.qerooy.base.dialect;

public class Dialect {

	public boolean supportsLimit() {
		return false;
	}

	public boolean supportsLimitOffset() {
		return supportsLimit();
	}

	public String getLimitString(String sql, int offset, int limit) {
		return getLimitString(sql, offset, Integer.toString(offset), limit, Integer.toString(limit));
	}

	public String getLimitString(String sql, int offset, String offsetPlaceholder, int limit, String limitPlaceholder) {
		throw new UnsupportedOperationException("paged queries not supported");
	}

}
{% endhighlight %}

5、编写spring配置文件applicationContext.xml、jdbc.properties 

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

	<context:component-scan base-package="*" />
	
	<util:properties id="dataSourceProps" location="classpath:jdbc.properties" />

	<bean id="dataSource" class="org.logicalcobwebs.proxool.ProxoolDataSource">
		<property name="driver" value="#{dataSourceProps['jdbc.driverClassName']}" />
		<property name="driverUrl" value="#{dataSourceProps['jdbc.url']}" />
		<property name="user" value="#{dataSourceProps['jdbc.username']}" />
		<property name="password" value="#{dataSourceProps['jdbc.password']}" />
	</bean>

	<!-- 配置Mybatis -->
	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mapperLocations" value="classpath:map/*.xml" />
		<property name="plugins">
			<list>
				<ref bean="pagingPlugin" />
			</list>
		</property>
	</bean>

	<bean id="pagingPlugin" class="com.qerooy.base.plugin.PagingPlugin">
		<property name="dialectClass">
			<value>#{dataSourceProps['jdbc.dialect']}</value>
		</property>
	</bean>

	<bean name="mapperScannerConfigurer" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
		<property name="basePackage" value="com.qerooy.mapper" />
		<property name="sqlSessionFactory" ref="sqlSessionFactory" />
	</bean>

</beans>

{% endhighlight %}

Properties代码
{% highlight ruby %}
jdbc.username=root
jdbc.password=123jkl
jdbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/qerooy?useUnicode=true&characterEncoding=UTF-8
jdbc.dialect=com.qerooy.base.dialect.MySQLDialect
jdbc.initialPoolSize=10
jdbc.minPoolSize=2
jdbc.maxPoolSize=5
jdbc.maxIdleTime=60
jdbc.acquireIncrement=5
jdbc.idleConnectionTestPeriod=60
jdbc.acquireRetryAttempts=20
jdbc.breakAfterAcquireFailure=true
jdbc.maxStatements=0
jdbc.testConnectionOnCheckout=false

{% endhighlight %}
6、编写Service操作数据库类及测试代码 
{% highlight ruby %}
package com.qerooy.service;

import java.util.List;

import org.apache.ibatis.session.RowBounds;

import com.qerooy.model.User;
import com.qerooy.model.UserExample;

public interface UserService {

	public User findById(Integer id);
	
	public List<User> findByPage(UserExample example,RowBounds rowBounds);
	
}

{% endhighlight %}

{% highlight ruby %}
package com.qerooy.service.impl;

import java.util.List;

import org.apache.ibatis.session.RowBounds;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.qerooy.mapper.UserMapper;
import com.qerooy.model.User;
import com.qerooy.model.UserExample;
import com.qerooy.service.UserService;

@Service
public class UserServiceImpl implements UserService {

	@Autowired
	UserMapper userMapper;
	
	@Override
	public User findById(Integer id) {
		return userMapper.selectByPrimaryKey(id);
	}
	
	@Override
	public List<User> findByPage(UserExample example,RowBounds rowBounds){
		return userMapper.selectByExampleWithRowbounds(example,rowBounds);
	}

}
{% endhighlight %}


{% highlight ruby %}
package com.qerooy;

import java.util.List;

import org.apache.ibatis.session.RowBounds;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import com.qerooy.model.User;
import com.qerooy.model.UserExample;
import com.qerooy.service.UserService;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration({"classpath*:applicationContext*.xml"}) 
public class MybatisTest {
	
	@Autowired
	UserService userService;

	@Test
	public void findById() {
		User user = userService.findById(3);
		System.out.println(user);
	}
	
	@Test
	public void findByPage() {
		UserExample example = new UserExample();
		RowBounds rowBounds = new RowBounds(1,10);
		List<User> list = userService.findByPage(example,rowBounds);
		System.out.println(list);
	}

}


{% endhighlight %}

为方便查看sql语句，可配置以下log4j.properties

{% highlight ruby %}
log4j.rootCategory=INFO, CONSOLE
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=[%p] %c %m%n

log4j.logger.com.qerooy.mapper = DEBUG

{% endhighlight %}

end.

