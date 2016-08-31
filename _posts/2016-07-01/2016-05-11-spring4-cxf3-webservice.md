---
layout: post
title: Spring4 + cxf3 构建wsdl webservice
date:   2016-05-11 11:55:11
categories: [java,spring,webservice]
---

## Spring4 + cxf3 构建wsdl webservice

本文介绍通过spring4.0.2 + apache cxf3.0.3构建一个wsdl webservice服务。

项目的结构如下：

```markdown
jaxws-client:客户端
jaxws-interface:公共接口
jaxws-server:服务端
说明:客户端与服务端均使用公共接口中的定义
```

1、新建一个maven项目jaxws-interface

其中pom.xml如下

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>jaxws-interface</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	
	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
		<log4j.version>1.2.17</log4j.version>
		<cxf.version>3.0.2</cxf.version>
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
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-transports-http</artifactId>
            <version>${cxf.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.apache.cxf</groupId>
            <artifactId>cxf-rt-frontend-jaxws</artifactId>
            <version>${cxf.version}</version>
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

编写一个接口UserService.java及一个对象类User.java，代码如下

```markdown
package com.hode.ws.service;

import java.util.Date;

import javax.jws.WebService;

import com.hode.ws.mode.User;

@WebService
public interface UserService{

	public Date getDate();
	
	public User findUser(String name);
	
}

```

```markdown
package com.hode.ws.mode;

import java.util.Date;

public class User{

	private String name;
	
	private int age;
	
	private double height;
	
	private Date birthday;

	public String getName(){
		return name;
	}

	public void setName(String name){
		this.name = name;
	}

	public int getAge(){
		return age;
	}

	public void setAge(int age){
		this.age = age;
	}

	public double getHeight(){
		return height;
	}

	public void setHeight(double height){
		this.height = height;
	}

	public Date getBirthday(){
		return birthday;
	}

	public void setBirthday(Date birthday){
		this.birthday = birthday;
	}

	@Override
	public String toString(){
		return "User [name=" + name + ", age=" + age + ", height=" + height + ", birthday=" + birthday + "]";
	}
	
}

```

interface层编写完毕。

2、新建一个服务端项目，此服务端使用jetty作为server

pom.xml如下，注意其中依赖了jaxws-interface，实际上服务端是实现了interface中定义的接口逻辑。

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>jaxws-server</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<dependencies>
		<dependency>
			<groupId>com.hode</groupId>
			<artifactId>jaxws-interface</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.12</version>
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
			
			<plugin>
				<artifactId>maven-war-plugin</artifactId>
				<configuration>
					<version>3.0</version>
				</configuration>
			</plugin>
		</plugins>
	</build>
	
</project>
```

编写一个实现类，实现接口功能，定义为UserServiceImpl.java，代码如下

```markdown
package com.hode.ws.service.impl;

import java.util.Date;

import com.hode.ws.mode.User;
import com.hode.ws.service.UserService;

public class UserServiceImpl implements UserService{

	@Override
	public Date getDate(){
		return new Date();
	}

	@Override
	public User findUser(String name){
		User u = new User();
		u.setAge(1);
		u.setBirthday(new Date());
		u.setHeight(12.1);
		u.setName("张三");
		return u;
	}

}


```

再编写spring配置文件applicationContext.xml，log4j.properties，JettyServer.java文件，代码依次如下：

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:util="http://www.springframework.org/schema/util"
	xmlns:cxf="http://cxf.apache.org/core" xmlns:jaxws="http://cxf.apache.org/jaxws" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
        http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd
        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
           
    <context:component-scan base-package="com.hode" />
    
	<import resource="classpath:META-INF/cxf/cxf.xml" />
	<import resource="classpath:META-INF/cxf/cxf-servlet.xml" />
	
	<bean id="loggingInInterceptor" class="org.apache.cxf.interceptor.LoggingInInterceptor" />
	
	<bean id="loggingOutInterceptor" class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
	
	<cxf:bus>
		<cxf:inInterceptors>
			<ref bean="loggingInInterceptor" />
		</cxf:inInterceptors>
		<cxf:outInterceptors>
			<ref bean="loggingOutInterceptor" />
		</cxf:outInterceptors>
	</cxf:bus>
	
	<jaxws:server id="userService" address="/userService">
		<jaxws:serviceBean>
			<bean class="com.hode.ws.service.impl.UserServiceImpl" />
		</jaxws:serviceBean>
	</jaxws:server>
	
</beans>

```


```markdown
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n

log4j.logger.com.hode=DEBUG
```


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

最后在目录webapp/WEB-INF中编写web.xml，代码如下：

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">
	
	<display-name>jaxws-server</display-name>
	
	<servlet>
		<servlet-name>CXFService</servlet-name>
		<servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
		<init-param>
			<param-name>config-location</param-name>
			<param-value>classpath:applicationContext.xml</param-value>
		</init-param>
	</servlet>

	<servlet-mapping>
		<servlet-name>CXFService</servlet-name>
		<url-pattern>/ws/*</url-pattern>
	</servlet-mapping>
	
	
</web-app>
```

3、新建一个客户端项目，完成测试

编写pom.xml，可以看到client端同样是依赖了jaxws-interface项目，代码如下：

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>jaxws-client</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	
	<dependencies>
		<dependency>
			<groupId>com.hode</groupId>
			<artifactId>jaxws-interface</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
		
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.7.12</version>
		</dependency>
		
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>4.0.2.RELEASE</version>
			<scope>test</scope>
		</dependency>
		
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.11</version>
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
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
		</plugins>
	</build>
	
</project>
```

编写测试类进行测试，注意含BaseTest及JAXWSClientTest如下:

```markdown
package com.hode;


import org.apache.log4j.Logger;
import org.junit.runner.RunWith;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
public class BaseTest {

	protected Logger log = Logger.getLogger(getClass());
	
}



package com.hode;

import java.util.Date;

import org.junit.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;

import com.hode.ws.mode.User;
import com.hode.ws.service.UserService;

@ContextConfiguration(locations={"classpath*:applicationContext.xml"})
public class JAXWSClientTest extends BaseTest {
	
	@Autowired
	private UserService userService;
	
	@Test
	public void getDate(){
		Date date = userService.getDate();
		log.info(date);
	}
	
	@Test
	public void findUser(){
		User user = userService.findUser("李四");
		log.info(user);
	}
	

}

```

再编写两个spring配置文件applicationContext.xml及log4j.properties文件

applicationContext.xml如下 

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xmlns:util="http://www.springframework.org/schema/util"
	xmlns:cxf="http://cxf.apache.org/core" xmlns:jaxws="http://cxf.apache.org/jaxws" 
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
        http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.0.xsd
        http://cxf.apache.org/core http://cxf.apache.org/schemas/core.xsd
        http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
    
	<import resource="classpath:META-INF/cxf/cxf.xml" />
	<import resource="classpath:META-INF/cxf/cxf-servlet.xml" />
	
	<context:component-scan base-package="com.hode" />
	
	<bean id="loggingInInterceptor" class="org.apache.cxf.interceptor.LoggingInInterceptor" />
	
	<bean id="loggingOutInterceptor" class="org.apache.cxf.interceptor.LoggingOutInterceptor" />
	
	<cxf:bus>
		<cxf:inInterceptors>
			<ref bean="loggingInInterceptor" />
		</cxf:inInterceptors>
		<cxf:outInterceptors>
			<ref bean="loggingOutInterceptor" />
		</cxf:outInterceptors>
	</cxf:bus>
        
    <jaxws:client id="userService" serviceClass="com.hode.ws.service.UserService" 
        address="http://localhost/ws/userService"/>
        
</beans>
```

log4j.properties文件可参考上面的写法。

4、完成编写后，执行测试

先运行JettyServer.java中的main方法，启动服务端，再运行测试类JAXWSClientTest截取部分日志如下

```markdown
----------------------------
ID: 1
Address: http://localhost/ws/userService
Encoding: UTF-8
Http-Method: POST
Content-Type: text/xml; charset=UTF-8
Headers: {Accept=[*/*], Cache-Control=[no-cache], connection=[keep-alive], Content-Length=[197], content-type=[text/xml; charset=UTF-8], Host=[localhost], Pragma=[no-cache], SOAPAction=[""], User-Agent=[Apache CXF 3.0.2]}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:findUser xmlns:ns2="http://service.ws.hode.com/"><arg0>李四</arg0></ns2:findUser></soap:Body></soap:Envelope>
--------------------------------------
13333 [qtp2142734953-21] INFO  org.apache.cxf.services.UserServiceImplService.UserServiceImplPort.UserService  - Outbound Message
---------------------------
ID: 1
Response-Code: 200
Encoding: UTF-8
Content-Type: text/xml
Headers: {}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:findUserResponse xmlns:ns2="http://service.ws.hode.com/"><return><age>1</age><birthday>2016-04-20T21:42:10.274+08:00</birthday><height>12.1</height><name>张三</name></return></ns2:findUserResponse></soap:Body></soap:Envelope>
--------------------------------------
13606 [qtp2142734953-22] INFO  org.apache.cxf.services.UserServiceImplService.UserServiceImplPort.UserService  - Inbound Message
----------------------------
ID: 2
Address: http://localhost/ws/userService
Encoding: UTF-8
Http-Method: POST
Content-Type: text/xml; charset=UTF-8
Headers: {Accept=[*/*], Cache-Control=[no-cache], connection=[keep-alive], Content-Length=[163], content-type=[text/xml; charset=UTF-8], Host=[localhost], Pragma=[no-cache], SOAPAction=[""], User-Agent=[Apache CXF 3.0.2]}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:getDate xmlns:ns2="http://service.ws.hode.com/"/></soap:Body></soap:Envelope>
--------------------------------------
13625 [qtp2142734953-22] INFO  org.apache.cxf.services.UserServiceImplService.UserServiceImplPort.UserService  - Outbound Message
---------------------------
ID: 2
Response-Code: 200
Encoding: UTF-8
Content-Type: text/xml
Headers: {}
Payload: <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"><soap:Body><ns2:getDateResponse xmlns:ns2="http://service.ws.hode.com/"><return>2016-04-20T21:42:10.760+08:00</return></ns2:getDateResponse></soap:Body></soap:Envelope>
--------------------------------------

```


<h4><a href="/rar/jaxws.rar"><b>Demo代码下载</b></a></h4>


结束。