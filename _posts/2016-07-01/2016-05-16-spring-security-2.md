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
spring-security并不验证用户名，可从源码中看到(查看源代码验证密码部分，关键代码在
org.springframework.security.authentication.dao.DaoAuthenticationProvider.additionalAuthenticationChecks(UserDetails, UsernamePasswordAuthenticationToken)方法中)，此时会将admin
保存到session中。


根据上一篇所介绍的简单使用，本文在此基础上完成配置，文章后面将附上源码以供下载学习


1、其中pom.xml、log4j.properties、applicationContext.xml、applicationContext-mvc.xml、web.xml不作修改。


2、修改applicationContext-security.xml，增加配置过滤拦截器完成对请求的拦截，内容如下

applicationContext-security.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
	xmlns:beans="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
        http://www.springframework.org/schema/security http://www.springframework.org/schema/security/spring-security-3.2.xsd">

	<http auto-config="true" use-expressions="true" access-denied-page="/access/denied.do">
		<!-- 允许访问 -->
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
		
		<custom-filter ref="myFilterSecurityInterceptor" before="FILTER_SECURITY_INTERCEPTOR"/>
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
	
	<beans:bean id="myFilterSecurityInterceptor" class="com.hode.security.MyFilterSecurityInterceptor">
		<beans:property name="accessDecisionManager" ref="accessDecisionManager" />
		<beans:property name="securityMetadataSource" ref="securityMetadataSource" />
		<beans:property name="authenticationManager" ref="authenticationManager" />
	</beans:bean>
	
	<beans:bean name="accessDecisionManager" class="com.hode.security.MyAccessDecisionManager" />
	
	<beans:bean name="securityMetadataSource" class="com.hode.security.MySecurityMetadataSource" />
	
</beans:beans>
```

其中myFilterSecurityInterceptor即为拦截器，securityMetadataSource主要是加载权限信息，accessDecisionManager判断是否具有权限

3、分别编写MyFilterSecurityInterceptor.java、MyAccessDecisionManager.java、MySecurityMetadataSource.java三个类

MyFilterSecurityInterceptor.java

```markdown
package com.hode.security;

import java.io.IOException;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;

import org.springframework.security.access.SecurityMetadataSource;
import org.springframework.security.access.intercept.AbstractSecurityInterceptor;
import org.springframework.security.access.intercept.InterceptorStatusToken;
import org.springframework.security.web.FilterInvocation;
import org.springframework.security.web.access.intercept.FilterInvocationSecurityMetadataSource;

public class MyFilterSecurityInterceptor extends AbstractSecurityInterceptor implements Filter{

	private FilterInvocationSecurityMetadataSource securityMetadataSource;
	
	public FilterInvocationSecurityMetadataSource getSecurityMetadataSource() {
		return securityMetadataSource;
	}

	public void setSecurityMetadataSource(
			FilterInvocationSecurityMetadataSource securityMetadataSource) {
		this.securityMetadataSource = securityMetadataSource;
	}

	@Override
	public void destroy() {
		
	}

	//拦截
	@Override
	public void doFilter(ServletRequest request, ServletResponse response,
			FilterChain chain) throws IOException, ServletException {
		FilterInvocation fi = new FilterInvocation(request, response, chain);
		InterceptorStatusToken token = super.beforeInvocation(fi);
		try{
			fi.getChain().doFilter(fi.getRequest(), fi.getResponse());
		}catch(Exception e){
			e.printStackTrace();
		}finally{
			super.afterInvocation(token,null);
		}
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
		
	}

	@Override
	public Class<?> getSecureObjectClass() {
		return FilterInvocation.class;
	}

	@Override
	public SecurityMetadataSource obtainSecurityMetadataSource() {
		return securityMetadataSource;
	}

}

```


MyAccessDecisionManager.java

```markdown
package com.hode.security;

import java.util.Collection;
import java.util.Iterator;

import org.springframework.security.access.AccessDecisionManager;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.authentication.InsufficientAuthenticationException;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.GrantedAuthority;

public class MyAccessDecisionManager implements AccessDecisionManager {

	@Override
	public void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes)
			throws AccessDeniedException, InsufficientAuthenticationException {
		Iterator<ConfigAttribute> it = configAttributes.iterator();
		while(it.hasNext()){
			String resource = it.next().getAttribute();
			for(GrantedAuthority ga:authentication.getAuthorities()){
				if(resource.equals(ga.getAuthority())){
					return ;
				}
			}
		}
		throw new AccessDeniedException("无权访问");
	}

	@Override
	public boolean supports(ConfigAttribute attribute) {
		return true;
	}

	@Override
	public boolean supports(Class<?> clazz) {
		return true	;
	}

}

```


MySecurityMetadataSource.java

```markdown
package com.hode.security;

import java.util.ArrayList;
import java.util.Collection;
import java.util.HashMap;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;

import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.access.SecurityConfig;
import org.springframework.security.web.FilterInvocation;
import org.springframework.security.web.access.intercept.FilterInvocationSecurityMetadataSource;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;
import org.springframework.security.web.util.matcher.RequestMatcher;

public class MySecurityMetadataSource implements FilterInvocationSecurityMetadataSource{

	public static Map<RequestMatcher,Collection<ConfigAttribute>> requestMap = new HashMap<RequestMatcher,Collection<ConfigAttribute>>(); //权限信息
	
	static{
		//此处的关键点是为角色配置相应的权限
		//为了演示方便采取硬编码的方式完成
		
		//角色 test_role_admin test_role_normal 拥有的权限如下
		RequestMatcher rm = new AntPathRequestMatcher("/manage**");
		ConfigAttribute ca = new SecurityConfig("test_role_admin");
		ConfigAttribute ca2 = new SecurityConfig("test_role_normal");
		Collection<ConfigAttribute> value = new ArrayList<ConfigAttribute>();
		value.add(ca);
		value.add(ca2);
		requestMap.put(rm, value);
		
		//角色 test_role_admin 拥有权限 /admin**
		rm = new AntPathRequestMatcher("/admin**");
		ca = new SecurityConfig("test_role_admin");
		value = new ArrayList<ConfigAttribute>();
		value.add(ca);
		requestMap.put(rm, value);
				
		//角色 test_role_normal 拥有权限 /normal**
		rm = new AntPathRequestMatcher("/normal**");
		ca = new SecurityConfig("test_role_normal");
		value = new ArrayList<ConfigAttribute>();
		value.add(ca);
		requestMap.put(rm, value);
		
	}
	
	@Override
	public Collection<ConfigAttribute> getAttributes(Object object)
			throws IllegalArgumentException {
		final HttpServletRequest request = ((FilterInvocation) object).getRequest();
        for (Map.Entry<RequestMatcher, Collection<ConfigAttribute>> entry : requestMap.entrySet()) {
            if (entry.getKey().matches(request)) {
                return entry.getValue();
            }
        }
        return null;
	}

	@Override
	public Collection<ConfigAttribute> getAllConfigAttributes() {
		return null;
	}

	@Override
	public boolean supports(Class<?> clazz) {
		return true;
	}

}

```

<font color="red">注意：MySecurityMetadataSource.java中加载的权限在此采用了硬编码方式为角色授权，为了方便演示将requestMap设置成了静态的且为public。
此时test_role_admin角色是没有访问/normal**的权限的，后面简易动态添加此权限。
</font>

<font color="red">当然动态权限也有利有蔽，一旦系统被攻破，后果也不可预知。</font>


4、调整UserDetailsServiceImpl.java类，为不同的用户配置不同角色

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
		if("admin".equals(username)){
			authList.add(new SimpleGrantedAuthority("test_role_admin"));
		}else if("normal".equals(username)){
			authList.add(new SimpleGrantedAuthority("test_role_normal"));
		}else{
			authList.add(new SimpleGrantedAuthority("test_role"));
		}
		//MD5("1{hode}")=4005c7b6cfee541c1a1414730cc9a98f,可以由任何md5摘要生成工具生成，当然此值应从数据库读取出来完成判断
		user = new User(username,"4005c7b6cfee541c1a1414730cc9a98f", true, true, true, true,authList);
		return user;
	}

}

```


5、调整AccessController.java类，添加动态权限功能，在applicationContext-security.xml中已配置了/access/**为放行，代码如下

```markdown
package com.hode.controller;

import java.util.ArrayList;
import java.util.Collection;

import javax.servlet.http.HttpServletRequest;

import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.access.SecurityConfig;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;
import org.springframework.security.web.util.matcher.RequestMatcher;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.hode.security.MySecurityMetadataSource;

@Controller
@RequestMapping("access")
public class AccessController {

	@RequestMapping("login")
	public String login(HttpServletRequest request){
		return "login"; //返回login.jsp
	}
	
	@RequestMapping("denied")
	public String denied(HttpServletRequest request){
		return "denied";
	}
	
	//为了简单起见,在此处动态配置admin访问normal.do权限
	@RequestMapping("reload")
	@ResponseBody
	public String reload(){
		RequestMatcher rm = new AntPathRequestMatcher("/normal**");
		ConfigAttribute ca = new SecurityConfig("test_role_admin");
		
		Collection<ConfigAttribute> value = null;
		if(MySecurityMetadataSource.requestMap.get(rm)!=null){
			value = MySecurityMetadataSource.requestMap.get(rm);
		}else{
			value = new ArrayList<ConfigAttribute>();
		}
		
		value.add(ca);
		MySecurityMetadataSource.requestMap.put(rm, value);
		return "reload success";
	}

}

```


<font color="red">reload方法中动态把权限添加到变量requestMap中了，实际使用中可从数据库、缓存、文件等读取。</font>

6、另外还有一些辅助类及页面，AdminController.java、NormalController.java，分别跳转到指定的页面，表示admin用户和normal用户可访问的请求。

AdminController.java

```markdown
package com.hode.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("admin")
public class AdminController {

	@RequestMapping("")
	public String admin(){
		return "admin";
	}
	
}

```

NormalController.java

```markdown
package com.hode.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("normal")
public class NormalController {

	@RequestMapping("")
	public String normal(){
		return "normal"; //返回normal.jsp
	}
	
}

```


<font color="red">具体jsp页面只作显示，可参看文章尾部demo下载</font>

页面说明

```markdown
admin.jsp：admin用户有权限访问显示的页面
denied.jsp：用户无权限访问显示的页面
login.jsp：用户登录显示的页面
manage.jsp：admin及normal用户都有权限访问显示的页面
normal.jsp：normal用户有权限访问显示的页面(注意：经过动态调整权限后，admin用户也可访问)
```


7、启动JettyServer，分别使用以下url访问完成动态权限测试

```markdown
打开登录页面 http://localhost/access/login.do
选中admin点击提交，此时admin用户登录成功；跳转到http://localhost/manage.do中，
根据权限配置admin用户有权访问http://localhost/admin.do，而无权限访问http://localhost/normal.do（访问时将显示禁止访问页面）

同样normal用户登录成功；跳转到http://localhost/manage.do中，
根据权限配置normal用户无权访问http://localhost/admin.do，而有权限访问http://localhost/normal.do

接着动态修改权限，使用admin登录成功，并访问http://localhost/access/reload.do将完成动态权限添加。此时admin用户已有权限访问http://localhost/normal.do

抛砖引玉，实际可根据业务场景完成对权限的增减。
```

<h4><a href="/rar/spring-security2.rar"><b>Demo代码下载</b></a></h4>


结束。
