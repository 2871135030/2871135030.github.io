---
layout: post
title: 使用Spring Session + Redis管理会话
date:   2016-05-13 11:14:48
categories: [spring,redis]
---

## 使用Spring Session + Redis管理会话

1、需先安装好redis，可以参考<a href="/tools/redis/2016/04/12/redis-install.html">redis安装</a>。

2、编写pom.xml文件，内容如下

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>spring-session</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>

	<properties>
		<spring.version>4.0.7.RELEASE</spring.version>
		<log4j.version>1.2.17</log4j.version>
		<jackson.version>1.9.12</jackson.version>
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
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
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
			<version>1.6.1</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.6.1</version>
		</dependency>
		
		<dependency>
		    <groupId>redis.clients</groupId>
		    <artifactId>jedis</artifactId>
		    <version>2.8.0</version>
		    <type>jar</type>
		</dependency>

		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-redis</artifactId>
			<version>1.6.1.RELEASE</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.session</groupId>
			<artifactId>spring-session</artifactId>
			<version>1.0.2.RELEASE</version>
		</dependency>

		<dependency>
			<groupId>org.codehaus.jackson</groupId>
			<artifactId>jackson-mapper-asl</artifactId>
			<version>${jackson.version}</version>
		</dependency>

		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
			<scope>test</scope>
		</dependency>

		<dependency>
			<groupId>javax.servlet</groupId>
			<artifactId>servlet-api</artifactId>
			<version>2.5</version>
			<scope>provided</scope>
		</dependency>

		<dependency>
			<groupId>org.eclipse.jetty</groupId>
			<artifactId>jetty-server</artifactId>
			<version>8.0.0.M3</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.eclipse.jetty</groupId>
			<artifactId>jetty-webapp</artifactId>
			<version>8.0.0.M3</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.eclipse.jetty</groupId>
			<artifactId>jetty-servlet</artifactId>
			<version>8.0.0.M3</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.eclipse.jetty</groupId>
			<artifactId>jetty-io</artifactId>
			<version>8.0.0.M3</version>
			<scope>provided</scope>
		</dependency>
		<dependency>
			<groupId>org.eclipse.jetty</groupId>
			<artifactId>jetty-util</artifactId>
			<version>8.0.0.M3</version>
			<scope>provided</scope>
		</dependency>

		<dependency>
			<groupId>org.mortbay.jetty</groupId>
			<artifactId>jsp-2.1-glassfish</artifactId>
			<version>2.1.v20100127</version>
			<scope>provided</scope>
		</dependency>

	</dependencies>

	<build>
		<plugins>
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
	</build>

</project>
```

3、编写web.xml，并保存到webapp/WEB-INF文件夹中

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">

	<display-name>spring-session</display-name>
	
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath*:applicationContext.xml</param-value>
	</context-param>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	
	<filter>
		<filter-name>springSessionRepositoryFilter</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>springSessionRepositoryFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>

	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
	<servlet>
		<servlet-name>DispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value></param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>DispatcherServlet</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>

</web-app>
```

4、编写spring配置文件，log4j及redis配置文件，分别如下

applicationContext.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<context:component-scan base-package="com.hode" />

	<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath*:redis.properties</value>
            </list>
        </property>
    </bean>
    
	<bean id="jedisPoolConfig" class="redis.clients.jedis.JedisPoolConfig"></bean>

	<bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
		<property name="hostName" value="${redis.host}" />
		<property name="port" value="${redis.port}" />
		<property name="password" value="${redis.pass}" />
		<property name="timeout" value="${redis.timeout}" />
		<property name="poolConfig" ref="jedisPoolConfig" />
		<property name="usePool" value="true" />
	</bean>

	<bean id="redisTemplate" class="org.springframework.data.redis.core.StringRedisTemplate">
		<property name="connectionFactory" ref="jedisConnectionFactory" />
	</bean>

	<!-- 将session放入redis -->
	<bean id="redisHttpSessionConfiguration" class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration" >
		<property name="maxInactiveIntervalInSeconds" value="1800" />
	</bean>

	<import resource="applicationContext-mvc.xml" />

</beans>
```

applicationContext-mvc.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
	
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
	
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/" />
		<property name="suffix" value=".jsp" />
		<property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
	</bean>

</beans>
```

log4j.properties

```markdown
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r %d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n

log4j.logger.com.hode =DEBUG
```

redis.properties

```markdown
redis.host=192.168.88.102
redis.port=6379
redis.pass=redis2016
redis.timeout=10000
```

5、编写Controller，分别编写登录，退出及查看当前登录状态Controller

LoginController.java

```markdown
package com.hode.controller;

import java.io.PrintWriter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.log4j.Logger;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("login")
public class LoginController {
	
	private Logger log = Logger.getLogger(getClass());

	@RequestMapping("")
	public void login(String name,String password,HttpServletRequest request,HttpServletResponse response) throws Exception{
		log.info("name : "+name);
		log.info("password : "+password);
		request.getSession().setAttribute("name", name);
		
		//输出
		PrintWriter writer = response.getWriter();
		writer.write("this is login");
		writer.flush();
		writer.close();
	}
	
}

```

LogoutController.java

```markdown
package com.hode.controller;

import java.io.PrintWriter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.log4j.Logger;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("logout")
public class LogoutController {
	
	private Logger log = Logger.getLogger(getClass());

	@RequestMapping("")
	public void logout(HttpServletRequest request,HttpServletResponse response) throws Exception{
		log.info("logout");
		String name = (String)request.getSession().getAttribute("name");
		log.info(name +" will logout");
		request.getSession().invalidate();
		
		//输出
		PrintWriter writer = response.getWriter();
		writer.write("this is logout");
		writer.flush();
		writer.close();
	}
	
}

```

MainController.java

```markdown
package com.hode.controller;

import java.io.PrintWriter;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.util.StringUtils;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("main")
public class MainController {
	
	@RequestMapping("")
	public void main(HttpServletRequest request,HttpServletResponse response) throws Exception{
		String name = (String)request.getSession().getAttribute("name");
		String message = "";
		if(StringUtils.isEmpty(name)){
			message = "no login";
		}else{
			message = "this is "+name+" login";
		}
		
		//输出
		PrintWriter writer = response.getWriter();
		writer.write(message);
		writer.flush();
		writer.close();
	}
	
}

```

6、编写JettyServer.java，完成测试

```markdown
package com.hode;

import org.eclipse.jetty.server.Server;
import org.eclipse.jetty.webapp.WebAppContext;


public class JettyServer {

	public static void main(String[] args) throws Exception{
		Server server = new Server(80);
		WebAppContext context = new WebAppContext();  
        context.setContextPath("/");  
        context.setWar("src/main/webapp");  
        server.setHandler(context);  
        server.start();  
        server.join();
	}

}

```


7、启动JettyServer，分别使用以下url访问

```markdown
登录
http://localhost/login.do?name=hode&password=123

查看登录状态
http://localhost/main.do

退出
http://localhost/logout.do
```


结束。
