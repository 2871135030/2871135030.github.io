---
layout: post
title: JDK动态代理及cglib动态代理实现分析
date:   2016-04-23 12:01:59
categories: [java]
---

## JDK动态代理及cglib动态代理实现分析

代理是一种常用的设计模式，其目的就是为其他对象提供一个代理以控制对某个对象的访问，动态代理使得开发人员无需手工编写代理类便可动态地获得代理类，下面就JDK动态代理与CGLIB动态代理展开分析。

1、JDK动态代理分析

JDK动态代理依靠接口实现，所以仅支持实现了接口的动态代理，下面用一个常用的JDK动态代理实现进行分析 

实现InvocationHandler实现调用处理器

InvocationHandlerImpl.java
{% highlight ruby %}
ppackage com.qerooy.handler;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

import org.apache.log4j.Logger;

/**
 * 动态代理类调用处理器
 */
public class InvocationHandlerImpl implements InvocationHandler {
	
	private Object target; //需代理的目录对象
	
	public InvocationHandlerImpl(Object target){
		this.target = target;
	}
	
	Logger log = Logger.getLogger(getClass());
	
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		log.info("InvocationHandlerImpl invoke start");
		Object obj = method.invoke(target, args);
		log.info("InvocationHandlerImpl invoke end");
		return obj;
	}
	
	/**
	 * 获取代理对象
	 * @param obj
	 * @return
	 */
	public Object get(Object obj){
		return java.lang.reflect.Proxy.newProxyInstance(obj.getClass().getClassLoader(),  
				obj.getClass().getInterfaces(), this);

	}

}

{% endhighlight %}

定义UserService接口，并实现此接口UserServiceImpl 
{% highlight ruby %}
package com.qerooy.service;

public interface UserService {
	
	public void saveUser();

}

{% endhighlight %}


UserServiceImpl.java
{% highlight ruby %}
package com.qerooy.service.impl;

import org.apache.log4j.Logger;

import com.qerooy.service.UserService;

public class UserServiceImpl implements UserService {

	Logger log = Logger.getLogger(getClass());
	
	public void saveUser() {
		log.info("this is saveUser");
	}

}

{% endhighlight %}

编写一个测试类进行测试 
JDKDynamicProxyTest.java
{% highlight ruby %}
package com.qerooy;

import org.apache.log4j.Logger;
import org.junit.Test;

import com.qerooy.handler.InvocationHandlerImpl;
import com.qerooy.service.UserService;
import com.qerooy.service.impl.UserServiceImpl;

public class JDKDynamicProxyTest {
	
	Logger log = Logger.getLogger(getClass());
	
	@Test
	public void test(){
		UserService userService = new UserServiceImpl();
		InvocationHandlerImpl handler = new InvocationHandlerImpl(userService);
		UserService service = (UserService)handler.get(userService);
		service.saveUser();
		
		log.info("生成的代理类名称:"+service.getClass().getName());
		log.info("test is ok");
	}

}

{% endhighlight %}

运行可得结果 
{% highlight ruby %}
INFO   InvocationHandlerImpl invoke start 
INFO   this is saveUser 
INFO   InvocationHandlerImpl invoke end 
INFO   生成的代理类名称:$Proxy4 
INFO   test is ok 
{% endhighlight %}

至此JDK动态代理实现完成。 

可以看到动态代理类是由java.lang.reflect.Proxy.newProxyInstance生成的，那么Proxy到底为我们生成了什么样的代理类呢？接下来通过源代码来了解一下Proxy到底是如何实现的，JDK的安装目录中均有源码src.zip。 
关键代码

{% highlight ruby %}
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException {
		// ... 省略
		//此处为生成代理类关键代码
		Class cl = getProxyClass(loader, interfaces);
		// ...省略
		Constructor cons = cl.getConstructor(constructorParams);
		return (Object) cons.newInstance(new Object[] { h });
		// ...省略
	}


public static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces) throws IllegalArgumentException {
		// ... 省略
		//此处为生成动态代理类的字节码,由defineClass0进行装载
		//所以我们可以用此方法生成代理类,将其反编译查看
		byte[] proxyClassFile = ProxyGenerator.generateProxyClass(proxyName, interfaces);
		// ... 省略
	}


{% endhighlight %}

由上可以看到生成代理类的方法，所以在Test类中将代理类生成，Test类改为

JDKDynamicProxyTest.java
{% highlight ruby %}
package com.qerooy;

import java.io.FileOutputStream;
import java.io.IOException;

import org.apache.log4j.Logger;
import org.junit.Test;

import sun.misc.ProxyGenerator;

import com.qerooy.handler.InvocationHandlerImpl;
import com.qerooy.service.UserService;
import com.qerooy.service.impl.UserServiceImpl;

public class JDKDynamicProxyTest {
	
	Logger log = Logger.getLogger(getClass());
	
	@Test
	public void test(){
		UserService userService = new UserServiceImpl();
		InvocationHandlerImpl handler = new InvocationHandlerImpl(userService);
		UserService service = (UserService)handler.get(userService);
		service.saveUser();
		
		log.info(service.getClass().getName());
		log.info("test is ok");
		//将生成的动态代理类保存到文件
		writeClassFile("c:/",service.getClass().getName(),userService.getClass().getInterfaces());
	}
	
	public static void writeClassFile(String path,String className,Class<?>[] clazz){
		//获取代理类的字节码
		byte[] classFile = ProxyGenerator.generateProxyClass(className,clazz);
		FileOutputStream out = null;
		try{
			out = new FileOutputStream(path+className+".class");
			out.write(classFile);
			out.flush();
		}catch(Exception e){
			e.printStackTrace();
		}finally{
			try{
				out.close();
			}catch(IOException e){
				e.printStackTrace();
			}
		}
	}

}


{% endhighlight %}

在C盘根目录下生成了动态的代理类class，使用反编译工具查看关键代码如下
{% highlight ruby %}
public final class $Proxy4 extends Proxy implements UserService
{

  public $Proxy4(InvocationHandler paramInvocationHandler)
    throws 
  {
    super(paramInvocationHandler);
  }

  public final void saveUser() throws 
  {
	  // ... 省略
      this.h.invoke(this, m3, null);//此处使用调用器方法并将方法名传回
      // ... 省略
  }
}

{% endhighlight %}

可以看到，生成的代理类继承了Proxy类并实现了UserService接口，而实现接口的方法中使用调用处理器的方法，处理器h则在构造方法中传入了return (Object) cons.newInstance(new Object[] { h }); 即生成的代理类中调用了InvocationHandler方法。 

简单来说生成的代理类中，每一个实现接口的方法均调用InvocationHandler的方法invoke，完成代理的逻辑。

2、CGLIB动态代理分析 

JDK动态代理只能代理实现了接口的类，若需代理的类未实现任何接口，则需要使用CGLIB生成代理类 
同样首先由简单的实现进行分析

编写一个未实现任何接口的类AccountServiceImpl 
AccountServiceImpl.java
{% highlight ruby %}
package com.qerooy.service.impl;
import org.apache.log4j.Logger;
public class AccountServiceImpl{
	Logger log = Logger.getLogger(getClass());
	
	public void saveAccount() {
		log.info("this is saveAccount");
	}
}

{% endhighlight %}

创建类MethodInterceptorImpl实现了CGLIB方法拦截器接口MethodInterceptor 
MethodInterceptorImpl.java
{% highlight ruby %}
package com.qerooy.interceptor;
import java.lang.reflect.Method;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import org.apache.log4j.Logger;
public class MethodInterceptorImpl implements MethodInterceptor{
	Logger log = Logger.getLogger(getClass());
	
	@Override
	public Object intercept(Object proxy, Method method, Object[] params, MethodProxy methodProxy) throws Throwable {
		log.info("MethodInterceptorImpl invoke start");
		Object object = methodProxy.invokeSuper(proxy, params);
		log.info("MethodInterceptorImpl invoke end");
		return object;
	}
	
	/**
	 * 获取代理对象
	 * @param obj
	 * @return
	 */
	public Object get(Class clazz){
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(clazz); //设置代理目标
        enhancer.setCallback(this); //设置回调
        enhancer.setClassLoader(clazz.getClassLoader());
        return enhancer.create();
	}
}

{% endhighlight %}

编写一个测试类
CGLIBDynamicProxyTest.java
{% highlight ruby %}
package com.qerooy;
import org.apache.log4j.Logger;
import org.junit.Test;
import com.qerooy.interceptor.MethodInterceptorImpl;
import com.qerooy.service.impl.AccountServiceImpl;
public class CGLIBDynamicProxyTest {
	Logger log = Logger.getLogger(getClass());
	
	@Test
	public void test(){
		MethodInterceptorImpl interceptor = new MethodInterceptorImpl();
		
		AccountServiceImpl service = (AccountServiceImpl)interceptor.get(AccountServiceImpl.class);
		log.info(service.getClass().getName());
		service.saveAccount();
		
		
		log.info("test is ok");
	}
}

{% endhighlight %}

运行结果如下

{% highlight ruby %}
INFO   MethodInterceptorImpl invoke start 
INFO   this is saveAccount 
INFO   MethodInterceptorImpl invoke end 
INFO   test is ok 
{% endhighlight %}

更新中。。。

结束。
