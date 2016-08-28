---
layout: post
title: Twitter的RPC框架Finagle简单使用介绍
date:   2015-11-15 13:00:48
categories: [java]
---

## Twitter的RPC框架Finagle简单使用介绍

本文接上篇<a href="/java/2015/11/12/apache-thrift.html">Apache Thrift使用介绍</a>，部分代码与上篇一样。


本例演示使用maven构建一个finagle简单例子，同样使用与上文相同的例子

1、编写pom.xml文件

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>finagle</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	
	<properties>
		<finagle.version>6.30.0</finagle.version>
		<scrooge.version>4.2.0</scrooge.version>
	</properties>
	
	<dependencies>
		<dependency>
			<groupId>com.twitter</groupId>
			<artifactId>scrooge-core_2.11</artifactId>
			<version>${scrooge.version}</version>
		</dependency>
		<dependency>
			<groupId>com.twitter</groupId>
			<artifactId>finagle-thrift_2.11</artifactId>
			<version>${finagle.version}</version>
		</dependency>

		<dependency>
			<groupId>org.apache.thrift</groupId>
			<artifactId>libthrift</artifactId>
			<version>0.5.0-1</version>
		</dependency>

		<dependency>
			<groupId>com.twitter</groupId>
			<artifactId>finagle-serversets_2.11</artifactId>
			<version>${finagle.version}</version>
			<exclusions>
				<exclusion>
					<groupId>org.slf4j</groupId>
					<artifactId>slf4j-jdk14</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>2.2.4</version>
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
			<!-- 注意这里使用twitter的插件 -->
			<plugin>
				<groupId>com.twitter</groupId>
				<artifactId>scrooge-maven-plugin</artifactId>
				<version>${scrooge.version}</version>
				<configuration>
					<thriftNamespaceMappings>
						<thriftNamespaceMapping><!-- 表示thrift文件中的映射,表示thrift映射包名为com.hode.thrift -->
							<from>thrift</from>
							<to>com.hode.thrift</to>
						</thriftNamespaceMapping>
					</thriftNamespaceMappings>
					<language>java</language> <!-- 生成语言,默认为scala -->
					<thriftOpts>
						<thriftOpt>--finagle</thriftOpt>
					</thriftOpts>
					<dependencyIncludes>
						<include>event-logger-thrift</include>
					</dependencyIncludes>
				</configuration>
				<executions>
					<execution>
						<id>thrift-sources</id>
						<phase>generate-sources</phase>
						<goals>
							<goal>compile</goal>
						</goals>
					</execution>
					<execution>
						<id>thrift-test-sources</id>
						<phase>generate-test-sources</phase>
						<goals>
							<goal>testCompile</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
			
		</plugins>
		<pluginManagement>
			<plugins>
				<!--This plugin's configuration is used to store Eclipse m2e settings 
					only. It has no influence on the Maven build itself. -->
				<plugin>
					<groupId>org.eclipse.m2e</groupId>
					<artifactId>lifecycle-mapping</artifactId>
					<version>1.0.0</version>
					<configuration>
						<lifecycleMappingMetadata>
							<pluginExecutions>
								<pluginExecution>
									<pluginExecutionFilter>
										<groupId>com.twitter</groupId>
										<artifactId>
											scrooge-maven-plugin
										</artifactId>
										<versionRange>[3.3.2,)</versionRange>
										<goals>
											<goal>compile</goal>
											<goal>testCompile</goal>
										</goals>
									</pluginExecutionFilter>
									<action>
										<ignore></ignore>
									</action>
								</pluginExecution>
							</pluginExecutions>
						</lifecycleMappingMetadata>
					</configuration>
				</plugin>
			</plugins>
		</pluginManagement>
	</build>
</project>
```

2、在maven目录src/main/中新建目录thrift，在此目录按thrift规范编写一个.thrift后缀的文件UserService.thift，内容如下

```markdown
/**定义包名,注意此处包名定义为thrift,其中pom.xml中作了映射 */
namespace java thrift

/**定义请求的传输对象*/
struct TUserRequest{
1:required string name,
2:required i32 age,
3:required i64 birthday,
4:required string remark
}

/**定义返回的传输对象*/
struct TUserResponse{
1:required string name,
2:required i32 age,
3:required i64 birthday,
4:required string remark
}

/**定义两个接口方法*/
service TUserService{
	void save(1:TUserRequest user)
	TUserResponse findByName(1:string name);
}
```

3、在项目根目录执行 mvn clean compile 命令，(twitter部分包可能不能正常通过maven下载，可自行网上查找下载)
便可以看到目录target\generated-sources\thrift\scrooge中生成了thrift源文件TUserService.java、TUserRequest.java、TUserResponse.java，
如果是在ide中编辑(如eclipse,netbeans)则直接将目录target\generated-sources\thrift\scrooge加到build path中。

4、接下来在src/main/java，创建实现类以及编写测试代码

实现类FUserServiceImpl.java

```markdown
package com.hode.finagle.impl;

import java.util.Date;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

import com.google.gson.Gson;
import com.hode.thrift.TUserRequest;
import com.hode.thrift.TUserResponse;
import com.hode.thrift.TUserService;
import com.twitter.util.ExecutorServiceFuturePool;
import com.twitter.util.Function0;
import com.twitter.util.Future;


public class FUserServiceImpl implements TUserService.ServiceIface{
	
	ExecutorService es = Executors.newFixedThreadPool(4); 
	
	ExecutorServiceFuturePool esfp = new ExecutorServiceFuturePool(es);

	Gson gson = new Gson(); //为了方便查看结果,使用此工具打印对象
	
	//保存对象测试
	@Override
	public Future<Void> save(final TUserRequest user) {
			
		Function0<Void> function = new Function0<Void>(){
			@Override
			public Void apply() {
				System.out.println(gson.toJson(user));
				System.out.println("save return void");
				return null;
			}
		};
		
		return esfp.apply(function);
	}

	//返回对象测试
	@Override
	public Future<TUserResponse> findByName(final String name) {
		
		System.out.println("start findByName...");
		System.out.println("name="+name);
		Function0<TUserResponse> function = new Function0<TUserResponse>(){

			@Override
			public TUserResponse apply() {
				TUserResponse rep = new TUserResponse();
		
				rep.setName("hode");
				rep.setAge(21);
				rep.setBirthday(new Date().getTime());
				rep.setRemark("测试备注");
				
				System.out.println(gson.toJson(rep));
				return rep;
			}
			
		};
		
		System.out.println("end findByName...");
		return esfp.apply(function);
		
	}

}

```

服务端代码UserServiceServer.java

```markdown
package com.hode.finagle;

import java.net.InetSocketAddress;

import org.apache.thrift.protocol.TBinaryProtocol;

import com.hode.finagle.impl.FUserServiceImpl;
import com.hode.thrift.TUserService;
import com.twitter.finagle.builder.ServerBuilder;
import com.twitter.finagle.thrift.ThriftServerFramedCodec;

public class UserServiceServer {

	public static void main(String[] args) {
		TUserService.ServiceIface server = new FUserServiceImpl();
		
		ServerBuilder.safeBuild(
				new TUserService.Service(server,new TBinaryProtocol.Factory()),
				ServerBuilder.get().name("UserServiceServer").codec(ThriftServerFramedCodec.get())
				.bindTo(new InetSocketAddress("localhost",8080)));
		
		System.out.println("UserServiceServer is start");
	}

}

```

客户端代码UserServiceClient.java，分别调用两个方法

```markdown
package com.hode.finagle;

import java.net.InetSocketAddress;
import java.util.Date;

import org.apache.thrift.protocol.TBinaryProtocol;

import com.hode.thrift.TUserRequest;
import com.hode.thrift.TUserResponse;
import com.hode.thrift.TUserService;
import com.twitter.finagle.Service;
import com.twitter.finagle.builder.ClientBuilder;
import com.twitter.finagle.thrift.ThriftClientFramedCodec;
import com.twitter.finagle.thrift.ThriftClientRequest;
import com.twitter.util.Await;
import com.twitter.util.Future;
import com.twitter.util.FutureEventListener;

public class UserServiceClient {

	public static void main(String[] args) throws Exception{
		Service<ThriftClientRequest,byte[]> client = ClientBuilder.safeBuild(
				ClientBuilder.get().hosts(new InetSocketAddress("localhost",8080))
				.codec(ThriftClientFramedCodec.get())
				.hostConnectionLimit(100));
		
		TUserService.ServiceIface userClient = new TUserService.ServiceToClient(client,new TBinaryProtocol.Factory());
		
		TUserRequest user = new TUserRequest();
		user.setAge(21);
		user.setBirthday(new Date().getTime());
		user.setName("hode");
		user.setRemark("test");
		
		//调用保存方法
		Future<Void> future = userClient.save(user).addEventListener(new FutureEventListener<Void>(){

			@Override
			public void onFailure(Throwable arg0) {
				System.out.println("call fail because: "+arg0);
			}

			@Override
			public void onSuccess(Void arg0) {
				System.out.println("call success.");
			}
			
		});
		
		System.out.println(Await.result(future.liftToTry()));
		
		//调用查询方法
		Future<TUserResponse> fu = userClient.findByName("hode").addEventListener(new FutureEventListener<TUserResponse>(){

			@Override
			public void onFailure(Throwable arg0) {
				System.out.println("call fail because: "+arg0);
			}

			@Override
			public void onSuccess(TUserResponse arg0) {
				System.out.println("call success. result:"+arg0);
			}
			
		});
		
		System.out.println(Await.result(fu.liftToTry()));
		
		client.close();
	}

}

```


5、如果在ide中可直接分别运行服务端与客户端即可，也可分别使用以下命令完成测试

```markdown
启动服务端执行
mvn clean compile exec:java -Dexec.mainClass="com.hode.finagle.UserServiceServer"

新打开一个命令窗口执行(存在线程,使用以下方式完成测试)
mvn exec:exec -Dexec.executable="java" -Dexec.args="-classpath %classpath;target com.hode.finagle.UserServiceClient"
```


结束。
