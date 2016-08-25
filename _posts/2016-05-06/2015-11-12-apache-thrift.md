---
layout: post
title: Apache thrift软件框架简单使用介绍
date:   2015-11-12 12:10:48
categories: [java]
---

## Apache thrift软件框架简单使用介绍

<a href="http://thrift.apache.org/">Thrift</a>是一个软件框架，用来进行可扩展且跨语言的服务的开发。它结合了功能强大的软件堆栈和代码生成引擎，
以构建在 C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, Cocoa, JavaScript, Node.js, Smalltalk, and OCaml 
等等编程语言间无缝结合的、高效的服务。

本例演示在windows中使用maven构建一个thrift简单例子，编写一个UserService实现保存及按姓名查询功能

1、编写pom.xml文件

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>thrift</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<dependencies>
		<dependency>
			<groupId>org.apache.thrift</groupId>
			<artifactId>libthrift</artifactId>
			<version>0.9.3</version>
		</dependency>
		
		<dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.5.8</version>
        </dependency>
        
       <dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>1.5.8</version>
		</dependency>

        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.14</version>
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
			<!-- apache thrift插件 其中thrift-0.9.3.exe直接从apache thrift官网下载便可 -->
			<plugin>
                <groupId>org.apache.thrift.tools</groupId>
                <artifactId>maven-thrift-plugin</artifactId>
                <version>0.1.11</version>
                <configuration>
                    <thriftExecutable>D:\thrift\thrift-0.9.3.exe</thriftExecutable>
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
										<groupId>org.apache.thrift.tools</groupId>
                						<artifactId>maven-thrift-plugin</artifactId>
										<versionRange>[0.1.11,)</versionRange>
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
/**定义包名 */
namespace java com.hode.thrift

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

3、在项目根目录执行 mvn clean compile 命令，便可以看到目录target\generated-sources\thrift中生成了thrift源文件TUserService.java、TUserRequest.java、TUserResponse.java，
如果是在ide中编辑(如eclipse,netbeans)则直接将目录target\generated-sources\thrift加到build path中。

4、接下来在src/main/java，创建实现类以及编写测试代码

编写TUserServiceImpl.java并实现接口TUserService.Iface，代码如下：

```markdown
package com.hode.thrift.impl;

import java.util.Date;

import org.apache.thrift.TException;

import com.google.gson.Gson;
import com.hode.thrift.TUserRequest;
import com.hode.thrift.TUserResponse;
import com.hode.thrift.TUserService;

public class TUserServiceImpl implements TUserService.Iface{

	Gson gson = new Gson(); //为了方便查看结果,使用此工具打印对象
	
	@Override
	public void save(TUserRequest user) throws TException {
		System.out.println(gson.toJson(user));
	}

	@Override
	public TUserResponse findByName(String name) throws TException {
		System.out.println("name="+name);
		TUserResponse rep = new TUserResponse();
		
		rep.setName("hode");
		rep.setAge(21);
		rep.setBirthday(new Date().getTime());
		rep.setRemark("测试备注");
		
		System.out.println(gson.toJson(rep));
		return rep;
	}
	
}

```


编写一个服务端代码

```markdown
package com.hode.thrift;

import org.apache.thrift.TProcessor;
import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TBinaryProtocol.Factory;
import org.apache.thrift.server.TServer;
import org.apache.thrift.server.TThreadPoolServer;
import org.apache.thrift.server.TThreadPoolServer.Args;
import org.apache.thrift.transport.TServerSocket;
import org.apache.thrift.transport.TTransportException;

import com.hode.thrift.impl.TUserServiceImpl;

public class UserServiceServer {

	public static void main(String[] args) {

		try{
			//设置服务端口为8080
			TServerSocket serverTransport = new TServerSocket(8080);
			//设置协议工厂为TBinaryProtocol.Factory()
			Factory proFactory = new TBinaryProtocol.Factory();
			//关联处理器与 Hello 服务的实现
			TProcessor processor = new TUserService.Processor<TUserService.Iface>(new TUserServiceImpl());
			
			Args arg = new TThreadPoolServer.Args(serverTransport).processor(processor);
			arg.protocolFactory(proFactory);
			TServer server = new TThreadPoolServer(arg);
			System.out.println("Start server on port 8080...");
			server.serve();
		}catch(TTransportException e){
			e.printStackTrace();
		}
		
	}

}

```

编写一个客户端代码

```markdown
package com.hode.thrift;

import java.util.Date;

import org.apache.thrift.protocol.TBinaryProtocol;
import org.apache.thrift.protocol.TProtocol;
import org.apache.thrift.transport.TSocket;
import org.apache.thrift.transport.TTransport;

import com.google.gson.Gson;

public class UserServiceClient {

	public static void main(String[] args) {
		Gson gson = new Gson();
		try {
			//设置调用的服务地址为本地，端口为8080
			TTransport transport = new TSocket("localhost",8080);
			transport.open();
			
			//设置传输协议为 TBinaryProtocol
			TProtocol protocol = new TBinaryProtocol(transport);
			TUserService.Client client = new TUserService.Client(protocol);
			
			TUserRequest user = new TUserRequest();
			user.setAge(21);
			user.setBirthday(new Date().getTime());
			user.setName("hode");
			user.setRemark("test");
			
			System.out.println(gson.toJson(user));
			//调用服务的方法
			client.save(user);
			
			TUserResponse rep = client.findByName("hode");
			System.out.println(gson.toJson(rep));
			
			
			transport.close();
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}


```


5、如果在ide中可直接分别运行服务端与客户端即可，也可分别使用以下命令完成测试

```markdown
启动服务端执行
mvn clean compile exec:java -Dexec.mainClass="com.hode.thrift.UserServiceServer"

新打开一个命令窗口执行
mvn clean compile exec:java -Dexec.mainClass="com.hode.thrift.UserServiceClient"
```


结束。
