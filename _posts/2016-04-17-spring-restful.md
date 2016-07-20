---
layout: post
title: Springmvc构建restful webservice应用
date:   2016-04-17 23:13:34
categories: java
---

## Spring4 mvc构建restful webservice应用

本文演示使用spring mvc开发一个简单的restful webservice服务，本例使用Jetty作为调试测试，通过使用RestTemplate完成服务调用。

1、编写pom.xml，其中包括spring、json解析所需的包，pom.xml内容如下
{% highlight ruby %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>spring-rest</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
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
					<source>1.8</source>
					<target>1.8</target>
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
{% endhighlight %}

2、编写一个SpringMVC的Controller，本例中继承自行编写的一个BaseController用于处理通用异常，代码分别如下
BaseController.java
{% highlight ruby %}
package com.hode.rest;

import java.util.List;

import org.apache.log4j.Logger;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.validation.ObjectError;
import org.springframework.web.bind.annotation.ExceptionHandler;

public class BaseController {

	protected Logger log = Logger.getLogger(getClass());
	
	protected void bindingResultHandler(BindingResult result) throws Exception{
		if(result.getErrorCount()>0){
			List<ObjectError> errors = result.getAllErrors();
			for(ObjectError error:errors){
				String errorMsg = "";
				if(error instanceof FieldError){
					FieldError fe = (FieldError)error;
					errorMsg = String.format("%s:%s",fe.getField(),error.getDefaultMessage());
					log.error(errorMsg);
				}else{
					errorMsg = String.format("%s:%s",error.getCode(),error.getDefaultMessage());
					log.error(errorMsg);
				}
				throw new Exception(errorMsg);
			}
		}
	}
	
	@ExceptionHandler
	public Object handleException(Exception e){
		log.error(String.format("请求异常[%s]",e));
		return e;
	}

		
}
{% endhighlight %}
UserRestController.java，其中入参与出参均为字符串，如需使用对象，则此对象需序列化
{% highlight ruby %}
package com.hode.rest;

import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("user")
public class UserRestController extends BaseController {

	@RequestMapping("getUser")
	public String getUser(@RequestBody String username){
		return String.format("success[%s]",username);
	}

}

{% endhighlight %}

3、编写spring配置文件及日志文件，各文件内容如下：
applicationContext.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<context:component-scan base-package="com.hode" />

	<import resource="applicationContext-mvc.xml"/>

</beans>
{% endhighlight %}
applicationContext-mvc.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<mvc:annotation-driven content-negotiation-manager="contentNegotiationManager">
		<mvc:message-converters>
			<bean class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
				<property name="supportedMediaTypes">
					<list>
						<value>application/json;charset=UTF-8</value>
					</list>
				</property>
				<property name="objectMapper">
					<bean class="org.codehaus.jackson.map.ObjectMapper">
						<property name="serializationInclusion">
							<value type="org.codehaus.jackson.map.annotate.JsonSerialize.Inclusion">NON_NULL</value>
						</property>
					</bean>
				</property>
			</bean>
		</mvc:message-converters>
	</mvc:annotation-driven>
	
	<bean id="contentNegotiationManager" class="org.springframework.web.accept.ContentNegotiationManagerFactoryBean">
		<property name="favorPathExtension" value="false" />
		<property name="ignoreAcceptHeader" value="true" />
		<property name="defaultContentType" value="application/json" />
		<property name="useJaf" value="false" />
		<property name="mediaTypes">
			<map>
				<entry key="json" value="application/json" />
			</map>
		</property>
	</bean>

	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/" />
		<property name="suffix" value=".jsp" />
		<property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
	</bean>

</beans>
{% endhighlight %}
log4j.properties
{% highlight ruby %}
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r %d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n

log4j.logger.com.hode = DEBUG
{% endhighlight %}

4、编写web.xml放置在目录WEB-INF中，同时写一个Main函数启动服务JettyServer，代码如下
web.xml
{% highlight ruby %}
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">
	
	<display-name>spring-rest</display-name>
	
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
			<param-value>classpath:applicationContext.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>DispatcherServlet</servlet-name>
		<url-pattern>*.rest</url-pattern>
	</servlet-mapping>
	
		
</web-app>
{% endhighlight %}
JettyServer.java
{% highlight ruby %}
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

{% endhighlight %}

编写完成后，运行JettyServer中的main方法，即可启动服务，关键日志截取如下
{% highlight ruby %}
INFO  org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping  - Mapped "{[/user/getUser],methods=[],params=[],headers=[],consumes=[],produces=[],custom=[]}" onto public java.lang.String com.hode.rest.UserRestController.getUser(java.lang.String)
{% endhighlight %}

5、服务端完成后，再编写一个客户端完成测试，为了方便查看发送及接收的报文情况，添加拦截器代码，测试代码如下：
BaseTest.java
{% highlight ruby %}
package com.hode;

import java.io.BufferedInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.util.ArrayList;
import java.util.List;

import org.apache.log4j.Logger;
import org.codehaus.jackson.map.DeserializationConfig;
import org.codehaus.jackson.map.ObjectMapper;
import org.springframework.http.HttpRequest;
import org.springframework.http.client.BufferingClientHttpRequestFactory;
import org.springframework.http.client.ClientHttpRequestExecution;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.ClientHttpRequestInterceptor;
import org.springframework.http.client.ClientHttpResponse;
import org.springframework.http.client.SimpleClientHttpRequestFactory;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.http.converter.json.MappingJacksonHttpMessageConverter;
import org.springframework.web.client.RestTemplate;

@SuppressWarnings("deprecation")
public abstract class BaseTest{

	protected String context = "http://localhost/";
	
	protected Logger log = Logger.getLogger(getClass());
	
	public RestTemplate getRT(){
		RestTemplate rt = new RestTemplate();
		ClientHttpRequestInterceptor interceptor = new ClientHttpRequestInterceptor(){

			@Override
			public ClientHttpResponse intercept(HttpRequest request,byte[] body,ClientHttpRequestExecution execution) throws IOException{
				log.info(request.getURI());
				log.info(request.getHeaders());
				log.info(new String(body));
				ClientHttpResponse response = execution.execute(request,body);
				
				InputStream input = response.getBody();
				BufferedInputStream bis = new BufferedInputStream(input);
				byte[] b = new byte[1024];
				int len = 0;
				StringBuffer buf = new StringBuffer();
				while((len=bis.read(b))!=-1){
					buf.append(new String(b,0,len));
				}
				bis.close();
				input.close();
				log.info(buf.toString());
				
				return response;
			}
			
		};
		rt.getInterceptors().add(interceptor);
		
		
		SimpleClientHttpRequestFactory requestFactory = new SimpleClientHttpRequestFactory();
		requestFactory.setConnectTimeout(5000);
		requestFactory.setReadTimeout(5000);
		
		ClientHttpRequestFactory bufRequestFactory = new BufferingClientHttpRequestFactory(requestFactory);
		
		
		rt.setRequestFactory(bufRequestFactory);
		
		MappingJacksonHttpMessageConverter jacksonConverter = new MappingJacksonHttpMessageConverter();
		ObjectMapper objectMapper = new ObjectMapper();
		objectMapper.configure(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES,false);
		jacksonConverter.setObjectMapper(objectMapper);
		
		List<HttpMessageConverter<?>> messageConverters = new ArrayList<HttpMessageConverter<?>>();
		messageConverters.add(jacksonConverter);
		rt.setMessageConverters(messageConverters);
		return rt;
	}
	
}

{% endhighlight %}
UserRestTest.java
{% highlight ruby %}
package com.hode;

import org.junit.Test;


public class UserRestTest extends BaseTest{

	@Test
	public void getUser(){
		
		String response = getRT().postForObject(context+"user/getUser.rest","test",String.class);
		System.out.println(response);
		System.out.println("success");
	}
	
}

{% endhighlight %}

先启动服务端，再运行单元测试UserRestTest，查看发送情况，截取部分关键日志如下：
{% highlight ruby %}
INFO  com.hode.UserRestTest  - http://localhost/user/getUser.rest
INFO  com.hode.UserRestTest  - {Accept=[application/json, application/*+json], Content-Type=[application/json;charset=UTF-8], Content-Length=[6]}
INFO  com.hode.UserRestTest  - "test"
INFO  com.hode.UserRestTest  - "success[test]"
{% endhighlight %}

结束。
