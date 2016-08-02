---
layout: post
title: Java代码保护，JS代码压缩工具
date:   2016-04-27 10:33:44
categories: [maven,tools]
---

## Java代码保护，JS代码压缩工具

作为软件公司，我们经常会将整个应用交由客户的运维人员在客户自己的内部服务器上部署运行，通常可能大部分程序或代码对客户可见并无大碍，
但针对软件公司自行研发的一些核心组件，以及使公司拥有持续的竞争力的技术积累或业务积累有关关键程序或代码，我们就得做一些必要的保护，
使得程序能正常运行，但代码却非常难懂。

本文介绍java代码保护工具proguard，以及js,css代码压缩工具yuicompressor。

1、建立2个maven项目，分别为hode-base、hode-web，hode-web依赖hode-base，而hode-base里主要编写java代码，hode-web中编写js,css代码，
最终打包项目查看代码保护结果。

2、先编写hode-base，缩写完成后可以查看代码保护的结果，编写pom.xml代码如下

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>hode-base</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<!-- 这里为java代码保护插件 -->
			<plugin>
                <groupId>com.github.wvengen</groupId>
                <artifactId>proguard-maven-plugin</artifactId>
                <version>2.0.7</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>proguard</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <attach>true</attach>
                    <attachArtifactClassifier>pg</attachArtifactClassifier>
                    <options>
                        <!--
                        <option>-dontobfuscate</option>
                        -->
                        <option>-ignorewarnings</option>
                        <option>-dontshrink</option>
                        <option>-dontoptimize</option>
                        <option>-dontusemixedcaseclassnames</option>
                        <option>-dontskipnonpubliclibraryclasses</option>
                        <option>-dontskipnonpubliclibraryclassmembers</option>
                        <!-- 处理后的包路径 -->
                        <option>-repackageclasses com.hode</option>

                        <option>-keepattributes Signature</option>
                        <option>-keepattributes *Annotation*</option>
                        <option>-keepattributes Exceptions</option>
                        <!-- 排除一些不需要处理的 -->
                        <option>-keep class com.hode.model.*{*;}</option>

                    </options>
                    <libs>
                        <lib>${java.home}/lib/rt.jar</lib>
                    </libs>

                </configuration>
            </plugin>
		</plugins>
	</build>
	
</project>
```

编写一个User.java，pom.xml中标识为排除，即有些vo类、model类或数据库映射类我们并不希望加密保护，我们可以排除

```markdown
package com.hode.model;

//这个类简单演示不被代码保护
public class User {

	private String name;

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}
	
}

```

编写一个操作类及接口，这2个生成的class将被密码保护

```markdown
package com.hode.service;

//简单写一个接口
public interface UserService {

	public boolean check();
	
}

package com.hode.service.impl;

import com.hode.service.UserService;

//简单写一个类，代码保护以后查看对应的方法情况
public class UserServiceImpl implements UserService{

	protected String nick;
	
	@Override
	public boolean check() {
		return false;
	}

}
```

完成编辑后，在hode-base项目中maven install完成打包安装，可到target目录中查看生成的jar包hode-base-0.0.1-SNAPSHOT-pg.jar
查看结果截图如下
![proguard，yuicompressor](/assets/dfeeded7-8f45-3579-9195-e0e6933a8a11.png)
![proguard，yuicompressor](/assets/dfeeded7-8f45-3579-9195-e0e6933a8a12.png)
![proguard，yuicompressor](/assets/dfeeded7-8f45-3579-9195-e0e6933a8a13.png)
![proguard，yuicompressor](/assets/dfeeded7-8f45-3579-9195-e0e6933a8a14.png)
![proguard，yuicompressor](/assets/dfeeded7-8f45-3579-9195-e0e6933a8a15.png)


可以看到User.java没有变化，而UserService.java，UserServiceImpl.java分别为a.class,b.class，其中里面的方法名及
属性名也已变更。


3、编写hode-web，缩写完成后可以查看代码保护的结果，编写pom.xml代码如下

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>hode-web</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<packaging>war</packaging>
	
	<dependencies>
		<dependency>
			<groupId>com.hode</groupId>
			<artifactId>hode-base</artifactId>
			<version>0.0.1-SNAPSHOT</version>
		</dependency>
	</dependencies>
	
	<build>
		<finalName>hode-web</finalName>

		<pluginManagement>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-resources-plugin</artifactId>
				<version>2.6</version>
				<configuration>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.1</version>
				<configuration>
					<source>1.7</source>
					<target>1.7</target>
					<encoding>UTF-8</encoding>
				</configuration>
			</plugin>
			<!-- 以下为maven打包插件 -->
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-war-plugin</artifactId>
				<version>2.1.1</version>
				<configuration>
					<version>3.0</version>
					<warSourceExcludes>
						js/*.js,
						css/*.css
					</warSourceExcludes>
				</configuration>
			</plugin>
			<!-- 以下为js代码压缩工具 -->
			<plugin>
                <groupId>net.alchim31.maven</groupId>
                <artifactId>yuicompressor-maven-plugin</artifactId>
                <version>1.3.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>compress</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <encoding>UTF-8</encoding>
                    <jswarn>false</jswarn>
                    <force>false</force>
 					<nosuffix>true</nosuffix>
 					<!-- 此处可以写排除一些不被压缩的js -->
                    <excludes>
                        <exclude>ext/*.js</exclude>
                        <exclude>ext/*.css</exclude>
                    </excludes>
                </configuration>
            </plugin>
		</plugins>
		</pluginManagement>
	</build>
</project>
```

下面webapp目录编辑文件，webapp目录下创建WEB-INF目录，在此目录下新建web.xml确保web工程无错误
web.xml不需要含任务内容即可

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<web-app id="WebApp_ID" version="2.4"
	xmlns="http://java.sun.mrm/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.mrm/xml/ns/j2ee http://java.sun.mrm/xml/ns/j2ee/web-app_2_4.xsd">

</web-app>
```

webapp目录下分别创建子目录css（需要压缩的css文件）、js（需要压缩的js文件）、ext（放置一些外部js或css，不作压缩，如jquery ui所需js,css）

在css目录中创建test.css，在js目录中创建test.js，文件内容如下

test.css如下

```markdown
.test {
	font-size: 14px;
	font-weight: bold;
	padding: 5px 0;
	margin-bottom: 10px;
	border-bottom: 1px solid #ccc;
}
```

test.js如下

```markdown
function cal(price,total){
	return price*total;
}
```

同时为了演示外部不被压缩的js及css，我们将test.css和test.js拷贝一份到目录ext中，目录截图如下

![proguard，yuicompressor](/assets/dfeeded7-8f45-3579-9195-e0e6933a8a16.png)

打包查看最终压缩结果

执行命令 yuicompressor:compress

```markdown
mvn clean yuicompressor:compress package
```

在打好的包中hode-web.war查看结果，可以看到js目录中的test.js及css目录中的test.css均已压缩，且test.js已经混淆，ext目录中的2个文件并没有进行压缩

压缩后的js及css示例如下：

```markdown
function cal(b,a){return b*a
};
```

```markdown
.test{font-size:14px;font-weight:bold;padding:5px 0;margin-bottom:10px;border-bottom:1px solid #ccc}
```

结束。
