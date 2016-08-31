---
layout: post
title: Dubbox的rest风格调用例子
date:   2016-05-02 12:33:01
categories: [spring,dubbo,webservice]
---

## Dubbox的rest风格调用例子

Dubbo是一个被国内很多互联网公司广泛使用的开源分布式服务框架，即使从国际视野来看应该也是一个非常全面的SOA基础框架。
作为一个重要的技术研究课题，在当当网我们根据自身的需求，为Dubbo实现了一些新的功能，并将其命名为Dubbox（即Dubbo eXtensions）


本例中使用jetty server，若本例中其中使用某些alibaba的jar包无法在网上下载，可直接下载源码
git clone https://github.com/dangdangdotcom/dubbox 将其安装到本地库中。


1、编写pom.xml，内容较多，其中包括jetty等

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>dubbox</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<properties>
		<spring.version>4.0.2.RELEASE</spring.version>
		<log4j.version>1.2.17</log4j.version>
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
            <groupId>javax.ws.rs</groupId>
            <artifactId>javax.ws.rs-api</artifactId>
            <version>2.0.1</version>
        </dependency>
        <dependency>
            <groupId>javax.annotation</groupId>
            <artifactId>javax.annotation-api</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>org.codehaus.jackson</groupId>
            <artifactId>jackson-mapper-asl</artifactId>
            <version>1.9.12</version>
        </dependency>
		
		<!--  dubbo start -->
		<dependency> 
		    <groupId>com.alibaba</groupId> 
		    <artifactId>dubbo</artifactId> 
		    <version>2.8.4</version> 
		    <exclusions> 
		        <exclusion> 
		            <groupId>org.springframework</groupId> 
		            <artifactId>spring</artifactId> 
		        </exclusion> 
		    </exclusions> 
		</dependency>
		
		<!--  dubbo end  -->
		
		<!-- zookeeper start --> 
		<dependency>
			<groupId>org.apache.zookeeper</groupId>
			<artifactId>zookeeper</artifactId>
			<version>3.5.0-alpha</version>
		</dependency>
		
		<dependency>
		    <groupId>com.101tec</groupId>
		    <artifactId>zkclient</artifactId>
		    <version>0.8</version>
		</dependency>
		<!-- zookeeper end -->
		
		<dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jaxrs</artifactId>
            <version>3.0.10.Final</version>
        </dependency>

        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-client</artifactId>
            <version>3.0.10.Final</version>
        </dependency>

        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-netty</artifactId>
            <version>3.0.10.Final</version>
        </dependency>

        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jdk-http</artifactId>
            <version>3.0.10.Final</version>
        </dependency>

        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jackson-provider</artifactId>
            <version>3.0.10.Final</version>
        </dependency>

        <dependency>
            <groupId>org.jboss.resteasy</groupId>
            <artifactId>resteasy-jaxb-provider</artifactId>
            <version>3.0.10.Final</version>
        </dependency>
        
        <dependency>
            <groupId>org.mortbay.jetty</groupId>
            <artifactId>jetty</artifactId>
            <version>6.1.26</version>
            <exclusions>
                <exclusion>
                    <groupId>org.mortbay.jetty</groupId>
                    <artifactId>servlet-api</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        
        <dependency>
            <groupId>javax.validation</groupId>
            <artifactId>validation-api</artifactId>
            <version>1.0.0.GA</version>
        </dependency>
		
		<!-- jetty begin -->
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
		<!-- jetty end -->
		
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

2、在目录webapp/WEB-INF中创建文件 web.xml，内容如下

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_ID" version="2.4"
	xmlns="http://java.sun.mrm/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.mrm/xml/ns/j2ee http://java.sun.mrm/xml/ns/j2ee/web-app_2_4.xsd">

	<display-name>dubbox restful</display-name>
    
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:applicationContext-provider.xml</param-value>
    </context-param>

    <!--this listener must be defined before the spring listener-->
    <listener>
        <listener-class>com.alibaba.dubbo.remoting.http.servlet.BootstrapListener</listener-class>
    </listener>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>com.alibaba.dubbo.remoting.http.servlet.DispatcherServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/services/*</url-pattern>
    </servlet-mapping>

</web-app>


```

3、编写服务接口DemoService，服务类DemoServiceImpl，一个VO类User以及JettyServer

```markdown
package com.hode.dubbo.provider;

import javax.ws.rs.Consumes;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import com.hode.model.User;

@Path("demo")
@Consumes({MediaType.APPLICATION_JSON, MediaType.TEXT_XML})
@Produces({MediaType.APPLICATION_JSON + ";"+MediaType.CHARSET_PARAMETER+"=UTF-8",MediaType.TEXT_XML+";"+MediaType.CHARSET_PARAMETER+"=UTF-8"})
public interface DemoService {

	@POST
    @Path("sayHello")
	public String sayHello(String name);
	
	@POST
    @Path("checkUser")
	public User checkUser(User u);
	
}

```

```markdown
package com.hode.dubbo.provider.impl;

import org.apache.log4j.Logger;

import com.hode.dubbo.provider.DemoService;
import com.hode.model.User;

public class DemoServiceImpl implements DemoService {

	private Logger log = Logger.getLogger(getClass());
	
	@Override
	public String sayHello(String name) {
		log.info("name = "+name);
		return "name is "+name;
	}

	@Override
	public User checkUser(User u) {
		log.info(u);
		u = new User();
		u.setName("hode");
		u.setAge(22);
		u.setHeight(168.0);
		return u;
	}

}

```

```markdown
package com.hode.model;

import java.io.Serializable;
import java.util.Date;

public class User implements Serializable{

	private static final long serialVersionUID = -4144338689776613101L;

	private String name;
	
	private int age;
	
	private double height;
	
	private Date birthday;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public double getHeight() {
		return height;
	}

	public void setHeight(double height) {
		this.height = height;
	}

	public Date getBirthday() {
		return birthday;
	}

	public void setBirthday(Date birthday) {
		this.birthday = birthday;
	}

	@Override
	public String toString() {
		return "User [name=" + name + ", age=" + age + ", height=" + height
				+ ", birthday=" + birthday + "]";
	}
	
}

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

4、编写spring配置文件applicationContext-provider.xml及log4j.properties，
其中使用了部分dubbox demo项目中的extension包内的类，可拷贝进去
代码如下

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
        http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd ">
	
	<!-- Provider信息 -->
	<dubbo:application name="dubbox-hode-provider" owner="programer" organization="dubbox" />
	
	<!-- 使用zookeeper注册中心暴露服务地址 -->
	<dubbo:registry address="zookeeper://192.167.48.128:2181" />
	
	<dubbo:annotation package="com.hode.dubbo.provider" />
	
	<!-- 定义rest协议,本例演示使用jetty -->
	<dubbo:protocol name="rest" port="8888" threads="500" contextpath="services" server="jetty" accepts="500"
                    extension="com.alibaba.dubbo.demo.extension.TraceInterceptor,
                    com.alibaba.dubbo.demo.extension.TraceFilter,
                    com.alibaba.dubbo.demo.extension.ClientTraceFilter,
                    com.alibaba.dubbo.demo.extension.DynamicTraceBinding,
                    com.alibaba.dubbo.demo.extension.CustomExceptionMapper,
                    com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter"/>
                    
	<!-- 声明需要暴露的服务接口 -->
	<dubbo:service interface="com.hode.dubbo.provider.DemoService" ref="demoService"  protocol="rest"/>
	
	<!-- 具体的实现bean -->
	<bean id="demoService" class="com.hode.dubbo.provider.impl.DemoServiceImpl" />
	
	
</beans>
```

```markdown
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r %d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n

log4j.logger.com.hode = DEBUG
log4j.logger.com.alibaba = DEBUG
```

5、最后编写一个测试类，使用resttemplate完成测试，测试代码如下


```markdown
package dubbox;

import org.springframework.http.HttpEntity;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.web.client.RestTemplate;

import com.hode.model.User;

public class RestClientTest {

	static RestTemplate rt = new RestTemplate();
	
	public static void main(String[] args) {
		sayHello();
		checkUser();
	}
	
	public static void checkUser(){
		String url = "http://localhost:8888/services/demo/checkUser.json";
		User u = new User();
		u.setName("test-hode");
		HttpHeaders headers = new HttpHeaders();
		headers.setContentType(MediaType.APPLICATION_JSON);
		HttpEntity request = new HttpEntity(u, headers);
		User result = rt.postForObject(url, request, User.class);
		
		System.out.println(result);
	}
	
	public static void sayHello(){
		String url = "http://localhost:8888/services/demo/sayHello.json";
		HttpHeaders headers = new HttpHeaders();
		headers.setContentType(MediaType.APPLICATION_JSON);
		HttpEntity request = new HttpEntity("hode", headers);
		String result = rt.postForObject(url, request, String.class);
		
		System.out.println(result);
	}

}

```

6、启动并测试


先运行JettyServer，再运行RestClientTest，查看结果

截取部分日志如下

```markdown
Request filter invoked
10965 2016-04-08 18:25:40,284 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The HTTP headers are: 
Accept: application/json, text/plain, application/json, application/*+json, */*
Accept-Charset: big5, big5-hkscs, cesu-8, euc-jp, euc-kr, gb18030, gb2312, gbk, ibm-thai, ibm00858, ibm01140, ibm01141, ibm01142, ibm01143, ibm01144, ibm01145, ibm01146, ibm01147, ibm01148, ibm01149, ibm037, ibm1026, ibm1047, ibm273, ibm277, ibm278, ibm280, ibm284, ibm285, ibm290, ibm297, ibm420, ibm424, ibm437, ibm500, ibm775, ibm850, ibm852, ibm855, ibm857, ibm860, ibm861, ibm862, ibm863, ibm864, ibm865, ibm866, ibm868, ibm869, ibm870, ibm871, ibm918, iso-2022-cn, iso-2022-jp, iso-2022-jp-2, iso-2022-kr, iso-8859-1, iso-8859-13, iso-8859-15, iso-8859-2, iso-8859-3, iso-8859-4, iso-8859-5, iso-8859-6, iso-8859-7, iso-8859-8, iso-8859-9, jis_x0201, jis_x0212-1990, koi8-r, koi8-u, shift_jis, tis-620, us-ascii, utf-16, utf-16be, utf-16le, utf-32, utf-32be, utf-32le, utf-8, windows-1250, windows-1251, windows-1252, windows-1253, windows-1254, windows-1255, windows-1256, windows-1257, windows-1258, windows-31j, x-big5-hkscs-2001, x-big5-solaris, x-euc-jp-linux, x-euc-tw, x-eucjp-open, x-ibm1006, x-ibm1025, x-ibm1046, x-ibm1097, x-ibm1098, x-ibm1112, x-ibm1122, x-ibm1123, x-ibm1124, x-ibm1166, x-ibm1364, x-ibm1381, x-ibm1383, x-ibm300, x-ibm33722, x-ibm737, x-ibm833, x-ibm834, x-ibm856, x-ibm874, x-ibm875, x-ibm921, x-ibm922, x-ibm930, x-ibm933, x-ibm935, x-ibm937, x-ibm939, x-ibm942, x-ibm942c, x-ibm943, x-ibm943c, x-ibm948, x-ibm949, x-ibm949c, x-ibm950, x-ibm964, x-ibm970, x-iscii91, x-iso-2022-cn-cns, x-iso-2022-cn-gb, x-iso-8859-11, x-jis0208, x-jisautodetect, x-johab, x-macarabic, x-maccentraleurope, x-maccroatian, x-maccyrillic, x-macdingbat, x-macgreek, x-machebrew, x-maciceland, x-macroman, x-macromania, x-macsymbol, x-macthai, x-macturkish, x-macukraine, x-ms932_0213, x-ms950-hkscs, x-ms950-hkscs-xp, x-mswin-936, x-pck, x-sjis_0213, x-utf-16le-bom, x-utf-32be-bom, x-utf-32le-bom, x-windows-50220, x-windows-50221, x-windows-874, x-windows-949, x-windows-950, x-windows-iso2022jp
Connection: keep-alive
Content-Length: 4
Content-Type: application/json
Host: localhost:8888
User-Agent: Java/1.8.0_66
, dubbo version: 2.8.4, current host: 127.0.0.1
Reader interceptor invoked
Dynamic reader interceptor invoked
10966 2016-04-08 18:25:40,285 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The contents of request body is: 
hode
, dubbo version: 2.8.4, current host: 127.0.0.1
10966 2016-04-08 18:25:40,285 [156710276@qtp-1193471756-492] INFO  com.hode.dubbo.provider.impl.DemoServiceImpl  - name = hode
10967 2016-04-08 18:25:40,286 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The HTTP headers are: 
Content-Type: application/json;charset=UTF-8
, dubbo version: 2.8.4, current host: 127.0.0.1
Response filter invoked
Writer interceptor invoked
Dynamic writer interceptor invoked
10967 2016-04-08 18:25:40,286 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The contents of response body is: 
name is hode
, dubbo version: 2.8.4, current host: 127.0.0.1
Request filter invoked
11066 2016-04-08 18:25:40,385 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The HTTP headers are: 
Accept: application/json, application/json, application/*+json
Connection: keep-alive
Content-Length: 57
Content-Type: application/json
Host: localhost:8888
User-Agent: Java/1.8.0_66
, dubbo version: 2.8.4, current host: 127.0.0.1
Reader interceptor invoked
Dynamic reader interceptor invoked
11069 2016-04-08 18:25:40,388 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The contents of request body is: 
{"name":"test-hode","age":0,"height":0.0,"birthday":null}
, dubbo version: 2.8.4, current host: 127.0.0.1
11070 2016-04-08 18:25:40,389 [156710276@qtp-1193471756-492] INFO  com.hode.dubbo.provider.impl.DemoServiceImpl  - User [name=test-hode, age=0, height=0.0, birthday=null]
11070 2016-04-08 18:25:40,389 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The HTTP headers are: 
Content-Type: application/json;charset=UTF-8
, dubbo version: 2.8.4, current host: 127.0.0.1
Response filter invoked
Writer interceptor invoked
Dynamic writer interceptor invoked
11072 2016-04-08 18:25:40,391 [156710276@qtp-1193471756-492] INFO  com.alibaba.dubbo.rpc.protocol.rest.support.LoggingFilter  -  [DUBBO] The contents of response body is: 
{"name":"hode","age":22,"height":168.0,"birthday":null}
, dubbo version: 2.8.4, current host: 127.0.0.1

```



<h4><a href="/rar/dubbox-restful.rar"><b>Demo代码下载</b></a></h4>

结束。