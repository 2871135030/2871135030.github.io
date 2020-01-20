---
layout: post
title: JDK8模拟jvm中Metaspace内存溢出(OutOfMemoryError)
date:   2021-11-12 14:14:00
categories: [jvm]
---

## JDK8模拟jvm中Metaspace内存溢出(OutOfMemoryError)

我们通过动态生成大量类来模拟Metaspace内存溢出


1、创建一个web项目

```
mvn archetype:generate -DgroupId=com.mixfate -DartifactId=metaspace -DarchetypeArtifactId=maven-archetype-webapp
```

2、修改pom.xml如下
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.mixfate</groupId>
  <artifactId>metaspace</artifactId>
  <packaging>war</packaging>
  <version>1.0.0</version>
  <name>metaspace Maven Webapp</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
	<dependency>
    <groupId>cglib</groupId>
    <artifactId>cglib</artifactId>
    <version>3.1</version>
</dependency>

  </dependencies>
  <build>
    <finalName>metaspace</finalName>
  </build>
</project>

```

3、编写模拟java类
```
package com.mixfate;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class Permanent{
	
	public static void main(String[] args){
		final String[] param = args;
		int i=0;
		while(true){
			System.out.println(i++);
			Enhancer enhancer = new Enhancer();
			enhancer.setSuperclass(Permanent.class);
			enhancer.setUseCache(false);
			enhancer.setCallback(new MethodInterceptor(){
				@Override
				public Object intercept(Object o,Method method,
				Object[] object,MethodProxy methodProxy) throws Throwable{
					return methodProxy.invokeSuper(o,param);
				}
			});
			enhancer.create();
		}
		
	}
	
}
```

4、执行以下命令即可模拟

```
mvn clean package exec:exec -Dexec.executable="java" -Dexec.args="-XX:MetaspaceSize=10M -XX:MaxMetaspaceSize=10M -classpath %classpath -verbose -verbose:gc  com.mixfate.Permanent"

```

结束。。。
