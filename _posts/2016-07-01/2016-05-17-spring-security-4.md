---
layout: post
title: Spring Security配置使用介绍（四）增加验证码
date:   2016-05-17 11:55:58
categories: [spring,java]
---

## Spring Security配置使用介绍（四）增加验证码

本文基于此前介绍过<a href="/spring/java/2016/05/16/spring-security-2.html">spring security动态配置权限</a>，使用的方法是增加过滤拦截对权限进行处理，
此篇介绍基于上一篇已实现动态权限配置，增加验证码功能。

1、实现spring-security验证码功能，只需要增加一个过滤器便可。此过滤继承UsernamePasswordAuthenticationFilter，在验证用户名密码之前验证码校验，
java代码实现如下

ValidateCodeAuthenticationFilter.java

```markdown
package com.hode.security;

import java.util.Date;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang.StringUtils;
import org.springframework.security.core.Authentication;
import org.springframework.security.core.AuthenticationException;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;
import org.springframework.security.web.authentication.session.SessionAuthenticationException;

public class ValidateCodeAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

	private final static String VALIDATECODE = "validateCode";

	@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response) throws AuthenticationException {
		System.out.println("开始校验-验证码-"+new Date());
		
		String requestCode = request.getParameter(VALIDATECODE);
		String sessionCode = (String) request.getSession().getAttribute(VALIDATECODE);
		request.getSession().removeAttribute(VALIDATECODE);
		if (!StringUtils.equalsIgnoreCase(requestCode, sessionCode)) {
			throw new SessionAuthenticationException("validatecode error");
		}else{
			//验证成功
			Authentication auth = super.attemptAuthentication(request, response);
			return auth;
		}
	}

}

```

可以看到在ValidateCodeAuthenticationFilter.java中，验证了一个字段为validateCode值是否与session中的值相同？判断验证码是否正确。
同时无论验证码是否验证正确，session中的值均需清除


2、增加一个生成图片验证码功能，此生成码的功能无需权限控制，将其添加到AccessControler.java中，使用了开源验证码生成工具patchca，demo中已附上源码。
PatchcaGenerate类请见demo或自行网上下载。

AccessController.java

```markdown
package com.hode.controller;

import java.util.ArrayList;
import java.util.Collection;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.patchca.PatchcaGenerate;
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
	
	@RequestMapping(value="validate")
	public void validate(HttpServletRequest req, HttpServletResponse res)
			throws Exception {
		res.setContentType("image/png");
		res.setHeader("Cache-Control", "no-cache, no-store");
		res.setHeader("Pragma", "no-cache");
		long time = System.currentTimeMillis();
		res.setDateHeader("Last-Modified", time);
		res.setDateHeader("Date", time);
		res.setDateHeader("Expires", time);

		HttpSession session = req.getSession(false);
		if (session == null) {
			session = req.getSession();
		}

		String validateCode = PatchcaGenerate.create(res.getOutputStream());
		session.setAttribute("validateCode", validateCode);
		
	}

}

```

3、在applicationContext-security.xml配置文件中增加一个filter配置即可

```markdown
<http ...
...
<custom-filter before="FORM_LOGIN_FILTER" ref="validateCodeAuthenticationFilter" />
...
</http>

<beans:bean id="validateCodeAuthenticationFilter" class="com.hode.security.ValidateCodeAuthenticationFilter" >
	<beans:property name="authenticationManager" ref="authenticationManager" />
</beans:bean>
	
...
```

4、login.jsp中增加一个图片验证码字段

```markdown
<div>验证码:<input type="text" name="validateCode" value=""/><img src="/access/validate.do" /></div>
```


5、启动JettyServer，分别使用以下url访问完成动态权限测试

```markdown
打开登录页面 http://localhost/access/login.do，注意：需使用验证码完成访问。
选中admin点击提交，此时admin用户登录成功；跳转到http://localhost/manage.do中，若验证码不正确，将提示验证失败。
```

<h4><a href="/rar/spring-security4.rar"><b>Demo代码下载</b></a></h4>


结束。
