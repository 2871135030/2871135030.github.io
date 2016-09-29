---
layout: post
title: Spring Security配置使用介绍（三）暴力动态配置权限（修改源码不推荐）
date:   2016-05-16 13:55:58
categories: [spring,java]
---

## Spring Security配置使用介绍（三）暴力动态配置权限（修改源码不推荐）

前面介绍过<a href="/spring/java/2016/05/16/spring-security-2.html">spring security动态配置权限</a>，使用的方法是增加过滤拦截对权限进行处理，
此篇介绍修改源码实现动态权限，部分代码基于上一篇进行修改，帮助理解spring security。

1、此方法修改源码，不需要另外增加拦截器，我们先在applicationContext-security.xml配置好已有权限，再动态调整其权限。

applicationContext-security.xml内容如下

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
		
		<!-- 已有权限 -->
		<intercept-url pattern="/admin**" access="hasRole('test_role_admin')" />
		<intercept-url pattern="/normal**" access="hasRole('test_role_normal')" />
		<intercept-url pattern="/manage**" access="hasAnyRole('test_role_admin','test_role_normal')" />
		
		<!-- 表示其他均需授权访问url -->
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

2、下载spring security源码，对类DefaultFilterInvocationSecurityMetadataSource进行修改，可以看到此类同样有个requestMap变量，为final私有变量，
将其final去掉并增加setter和getter方法，用于动态修改权限，修改后代码如下（仅列出变更处，详情可看demo，文章尾部附下载地址）

```markdown
......
	private Map<RequestMatcher, Collection<ConfigAttribute>> requestMap;
......
	public Map<RequestMatcher, Collection<ConfigAttribute>> getRequestMap() {
		return requestMap;
	}

	public void setRequestMap(
			Map<RequestMatcher, Collection<ConfigAttribute>> requestMap) {
		this.requestMap = requestMap;
	}
......	
```

3、新增一个SpringUtil.java工具类，用于取bean，

SpringUtil.java

```markdown
package com.hode.util;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class SpringUtil implements ApplicationContextAware {

	public static ApplicationContext context;

	public static ApplicationContext getApplicationContext() {
		return context;
	}

	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		SpringUtil.context = applicationContext;
	}
	
	@SuppressWarnings("unchecked")
	public static <T> T getBean(String name) throws BeansException{
		return (T)context.getBean(name);
	}
	
	@SuppressWarnings("unchecked")
	public static <T> T getBean(Class<?> clazz) throws BeansException{
		return (T)context.getBean(clazz);
	}

}
```

4、下载spring security源码，对ExpressionBasedFilterInvocationSecurityMetadataSource进行修改，将processMap方法改为公有，同时String expression下方增加判断，

修改后代码如下（仅列出变更处，详情可看demo，文章尾部附下载地址）

```markdown
......
 public static LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> processMap
......
if(expression==null) //注意首次expression若有值,attribute已转为WebExpressionConfigAttribute类型对象
            	expression = entry.getValue().toArray(new ConfigAttribute[1])[0].toString();
......
```

5、重新调整AccessController.java中的reload方法，内容如下

AccessController.java

```markdown
package com.hode.controller;

import java.util.ArrayList;
import java.util.Collection;
import java.util.LinkedHashMap;

import javax.servlet.http.HttpServletRequest;

import org.springframework.security.access.ConfigAttribute;
import org.springframework.security.access.SecurityConfig;
import org.springframework.security.web.access.expression.DefaultWebSecurityExpressionHandler;
import org.springframework.security.web.access.expression.ExpressionBasedFilterInvocationSecurityMetadataSource;
import org.springframework.security.web.access.intercept.FilterSecurityInterceptor;
import org.springframework.security.web.util.matcher.AntPathRequestMatcher;
import org.springframework.security.web.util.matcher.RequestMatcher;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

import com.hode.util.SpringUtil;

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
	
	//此处动态配置admin访问normal.do权限
	@RequestMapping("reload")
	@ResponseBody
	public String reload(){
		try{
			FilterSecurityInterceptor interceptor = SpringUtil.getBean(FilterSecurityInterceptor.class);
			DefaultWebSecurityExpressionHandler expressionHandler = SpringUtil.getBean(DefaultWebSecurityExpressionHandler.class);
			ExpressionBasedFilterInvocationSecurityMetadataSource metadataSource = (ExpressionBasedFilterInvocationSecurityMetadataSource) interceptor.obtainSecurityMetadataSource();
	
			LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> requestMap = (LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>>) metadataSource.getRequestMap();
	
			//修改权限
			RequestMatcher rm = new AntPathRequestMatcher("/normal**");
    		ConfigAttribute ca = new SecurityConfig("hasAnyRole('test_role_normal','test_role_admin')");
			Collection<ConfigAttribute> value = new ArrayList<ConfigAttribute>();
			value.add(ca);
			requestMap.put(rm, value);
			
	
			LinkedHashMap<RequestMatcher, Collection<ConfigAttribute>> result = ExpressionBasedFilterInvocationSecurityMetadataSource.processMap(requestMap, expressionHandler.getExpressionParser());
			metadataSource.setRequestMap(result);
		}catch(Exception e){
			e.printStackTrace();
		}
		return "reload success";
	}

}

```


6、启动JettyServer，分别使用以下url访问完成动态权限测试

```markdown
打开登录页面 http://localhost/access/login.do
选中admin点击提交，此时admin用户登录成功；跳转到http://localhost/manage.do中，
根据权限配置admin用户有权访问http://localhost/admin.do，而无权限访问http://localhost/normal.do（访问时将显示禁止访问页面）

同样normal用户登录成功；跳转到http://localhost/manage.do中，
根据权限配置normal用户无权访问http://localhost/admin.do，而有权限访问http://localhost/normal.do

接着动态修改权限，使用admin登录成功，并访问http://localhost/access/reload.do将完成动态权限添加。此时admin用户已有权限访问http://localhost/normal.do

```

<h4><a href="/rar/spring-security3.rar"><b>Demo代码下载</b></a></h4>


结束。
