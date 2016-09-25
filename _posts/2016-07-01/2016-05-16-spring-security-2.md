---
layout: post
title: Spring Security配置使用介绍（二）动态配置权限
date:   2016-05-16 12:55:58
categories: [spring,java]
---

## Spring Security配置使用介绍（二）动态配置权限

本文接上文<a href="/spring/java/2016/05/15/spring-security.html">Spring Security配置使用介绍（一）简单配置</a>，

由上一篇介绍，在配置文件applicationContext-security.xml中配置了intercept-url pattern="/**" access="isAuthenticated()"，表示需登录授权访问，
这是一个非常简单的权限控制，实际使用中我们需要增加角色并配置相应的权限。

在上文中的demo中的代码可以看到，login.jsp中定义了用户名为1，而在类UserDetailsServiceImpl中new User()中填入的用户名为admin，这主要是因为
spring-security并不验证用户名，可从源码中看到(查看源代码验证密码部分，并在此列出)


1、编写pom.xml，主要有spring及其security相关依赖

```markdown

```




2、编写Spring配置文件及log4j.properties文件，在此为了区分配置，Spring配置文件有3个，
applicationContext.xml为主配置文件，applicationContext-mvc.xml、applicationContext-security.xml分别配置mvc、security，
内容分别如下

applicationContext.xml

```markdown
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

	<context:component-scan base-package="com.hode" />
	
    <import resource="applicationContext-mvc.xml"/>
    
    <import resource="applicationContext-security.xml"/>

</beans>
```

applicationContext-mvc.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd">

	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping" />
	
	<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter" />
	
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">  
        <property name="prefix" value="/"/>  
        <property name="suffix" value=".jsp"/>  
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />  
    </bean>

</beans>
```

applicationContext-security.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-3.2.xsd">

	<http auto-config="true" use-expressions="true">
		<!-- 允许访问  -->
		<intercept-url pattern="/css/**" access="permitAll" />
		<intercept-url pattern="/js/**" access="permitAll" />
		<intercept-url pattern="/access/**" access="permitAll" />
		<!-- 需授权访问url -->
		<intercept-url pattern="/**" access="isAuthenticated()" />
		<!-- 登录url及登录失败显示url -->
		<form-login login-page="/access/login.do"
			authentication-failure-url="/access/login.do"
			authentication-success-handler-ref="loginSuccessHandler"/>
		<logout success-handler-ref="logoutSuccessHandler" />
	</http>
	
	<beans:bean id="loginSuccessHandler" class="com.hode.security.handler.UserLoginSuccessHandler">
		<beans:property name="defaultTargetUrl" value="/manage.do" />
	</beans:bean>
	
	<beans:bean id="logoutSuccessHandler" class="com.hode.security.handler.UserLogoutSuccessHandler">
		<beans:property name="defaultTargetUrl" value="/access/login.do" />
	</beans:bean>
	
	<authentication-manager alias="authenticationManager">
		<authentication-provider user-service-ref="userDetailsService">
			<password-encoder hash="md5" ><salt-source system-wide="hode" /></password-encoder>
		</authentication-provider>
	</authentication-manager>
	
</beans:beans>
```

log4j.properties

```markdown
log4j.rootLogger=INFO,Console
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=%-4r %d{yyyy-MM-dd HH:mm:ss,SSS} [%t] %-5p %c %x - %m%n

log4j.logger.com.hode =DEBUG
```

<font color="red">
注：请注意applicationContext-security.xml配置文件中的配置
<br>permitAll是允许访问
<br>isAuthenticated()表示需认证后访问，即只需登录成功就可访问，无关乎角色。
<br>UserLoginSuccessHandler与UserLogoutSuccessHandler分别为登录成功与退出成功处理，当然登录失败仍为登录页。
<br>authentication-manager标签设置用户名密码校验逻辑，password-encoder设置为md5，salt-source设置为hode，此时spring security
校验的密码流程：如用户提交的密码为1，则实际将生成样式如字符串1{hode}，再对此字符串使用md5生成摘要，所以在authentication-provider标签中定义的
userDetailsService中的密码即应该为MD5("1{hode}")的结果。其他密码如MD5("123456{hode}")，MD5("abcdef{hode}")
</font>


3、编写Controller

AccessController.java，如上面配置文件中设置了允许访问，在这里主要是登录页面。有其他如验证码生成等

```markdown
package com.hode.controller;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("access")
public class AccessController {

	@RequestMapping("login")
	public String login(HttpServletRequest request){
		return "login"; //返回login.jsp
	}

}
```

ManageController.java，如上面配置文件中未做特殊配置，需授权访问。

```markdown
package com.hode.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("manage")
public class ManageController {

	@RequestMapping("")
	public String manage(){
		return "manage"; //返回manage.jsp
	}
	
}
```

4、编写handler及用户登录处理

UserLoginSuccessHandler.java，用户登录成功处理的handler

```markdown
package com.hode.security.handler;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.SimpleUrlAuthenticationSuccessHandler;

/**
 * 用户登录成功后处理器
 */
public class UserLoginSuccessHandler extends SimpleUrlAuthenticationSuccessHandler {

	@Override
	public void onAuthenticationSuccess(HttpServletRequest req, HttpServletResponse res, Authentication au) throws IOException, ServletException {
		UserDetails user = (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
		logger.info("用户["+user.getUsername()+"]登录成功!");
		super.onAuthenticationSuccess(req, res, au);
	}

}
```

UserLogoutSuccessHandler.java，用户退出成功处理的handler

```markdown
package com.hode.security.handler;

import java.io.IOException;

import javax.servlet.ServletException;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.web.authentication.logout.SimpleUrlLogoutSuccessHandler;

public class UserLogoutSuccessHandler extends SimpleUrlLogoutSuccessHandler {

	@Override
	public void onLogoutSuccess(HttpServletRequest request, HttpServletResponse response, Authentication authentication) throws IOException, ServletException {
		UserDetails user = (UserDetails)authentication.getPrincipal();
		logger.info("用户["+user.getUsername()+"]退出成功!");
		super.onLogoutSuccess(request, response, authentication);
	}

}
```

UserDetailsServiceImpl.java，此类完成的任何即是用户赋权限判断，校验密码

```markdown
package com.hode.security.service;

import java.util.ArrayList;
import java.util.List;

import org.apache.log4j.Logger;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.stereotype.Service;

@Service("userDetailsService")
public class UserDetailsServiceImpl implements UserDetailsService {

	private Logger log = Logger.getLogger(getClass());
	
	@Override
	public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
		UserDetails user = null; 
		log.info("loadUserByUsername username:"+username);
		List<GrantedAuthority> authList = new ArrayList<GrantedAuthority>();
		authList.add(new SimpleGrantedAuthority("test_role")); //随意填写角色
		//MD5("1{hode}")=4005c7b6cfee541c1a1414730cc9a98f,可以由任何md5摘要生成工具生成，当然此值应从数据库读取出来完成判断
		user = new User("admin","4005c7b6cfee541c1a1414730cc9a98f", true, true, true, true,authList);
		return user;
	}

}
```

5、编写web.xml，登录页login.jsp及登录成功页manage.jsp

webapp目录下创建目录WEB-INF，此目录下新建web.xml文件，内容如下

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_ID" version="2.4"
	xmlns="http://java.sun.mrm/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.mrm/xml/ns/j2ee http://java.sun.mrm/xml/ns/j2ee/web-app_2_4.xsd">

	<display-name>spring-security</display-name>
	
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath*:applicationContext.xml</param-value>
	</context-param>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	
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

	<filter>
		<filter-name>springSecurityFilterChain</filter-name>
		<filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
	</filter>

	<filter-mapping>
		<filter-name>springSecurityFilterChain</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
	<servlet>
		<servlet-name>springSecurity</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value></param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>

	<servlet-mapping>
		<servlet-name>springSecurity</servlet-name>
		<url-pattern>*.do</url-pattern>
	</servlet-mapping>
	
</web-app>
```

登录页login.jsp

```markdown
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>登录页面</title>
    
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
  </head>
  
  <body>
  <form id="ff" method="post" action="${pageContext.request.contextPath}/j_spring_security_check">
	<div>登录页面</div>
	<div>用户名:<input type="text" name="j_username" value="1"/></div>
	<div>密码:<input type="text" name="j_password" value="1"/></div>
	<div><input type="submit" name="提交" /></div>
  </form>
    
  </body>
</html>

```

管理页manage.jsp

```markdown
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>管理页面</title>
    
	<meta http-equiv="pragma" content="no-cache">
	<meta http-equiv="cache-control" content="no-cache">
	<meta http-equiv="expires" content="0">    
  </head>
  
  <body>
    <div>这是一个管理页面</div>
	<div><a href="${pageContext.request.contextPath}/j_spring_security_logout">退出</a></div>
  </body>
</html>

```

<font color="red">
注：请注意使用spring security后，
<br>登录请求地址如${pageContext.request.contextPath}/j_spring_security_check，用户名、密码须为j_username、j_password
<br>退出请求地址如${pageContext.request.contextPath}/j_spring_security_logout
</font>


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
http://localhost/login.do

管理页
http://localhost/manage.do

```

<h4><a href="/rar/spring-security.rar"><b>Demo代码下载</b></a></h4>


本节结束，后面小节均以本例为基础。
