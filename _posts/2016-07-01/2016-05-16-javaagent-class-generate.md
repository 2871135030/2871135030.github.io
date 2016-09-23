---
layout: post
title: 使用javaagent参数保存二进制class文件
date:   2016-05-16 15:02:58
categories: [java]
---

## 使用javaagent参数保存二进制class文件

使用-javaagent 参数用户可以在执行main函数前执行一些其他操作，如可以动态的修改替换类中代码。既然可以修改类那么自然就可以获取到类（class二进制）了。
本文同样使用maven去打一个包

1、编写pom.xml，此处仅仅是定义一个jar工程

```markdown
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.hode</groupId>
	<artifactId>clazzagent</artifactId>
	<version>0.0.1-SNAPSHOT</version>

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
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-jar-plugin</artifactId>
				<version>2.4</version>
				<configuration>
					<archive><!-- 如果没有被正确打进jar包,请确认MANIFEST.MF是否正确,直接拷贝正确的自动生成的 -->
						<manifestFile>src/main/resources/META-INF/MANIFEST.MF</manifestFile>
					</archive>
				</configuration>
			</plugin>
		</plugins>
	</build>

</project>
```

2、编写一个导出class的类，此类实现了ClassFileTransformer接口并且实现了一个静态方法premain，为了简单起见，只保存指定包下的类二进制文件到指定的目录中。
代码如下
ClazzGenerateAgent .java

```markdown
package com.hode;

import java.io.BufferedOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.lang.instrument.ClassFileTransformer;
import java.lang.instrument.IllegalClassFormatException;
import java.lang.instrument.Instrumentation;
import java.security.ProtectionDomain;

/**
 * 导出生成类ClazzGenerateAgent
 * 需要实现一个静态的public static void premain(String agentArgs,Instrumentation inst)方法
 */
public class ClazzGenerateAgent implements ClassFileTransformer{

	public static void premain(String agentArgs,Instrumentation inst){
		inst.addTransformer(new ClazzGenerateAgent());
	}
	
	@Override
	public byte[] transform(ClassLoader loader, String className,
			Class<?> classBeingRedefined, ProtectionDomain protectionDomain,
			byte[] classfileBuffer) throws IllegalClassFormatException {
		//只导出指定包名下的类
		if(className.contains("com/hode")){
			System.out.println(className);
			int lastIndexOf = className.lastIndexOf("/") + 1;
			String fileName = className.substring(lastIndexOf) + ".class";
			exportClazz("c:/clazz/", fileName, classfileBuffer);
			System.out.println(className+" export success!");
		}else{
			
		}
		return null;
	}
	
	/**
	 * 将data保存到文件中
	 * @param path
	 * @param filename
	 * @param data
	 */
	private void exportClazz(String path,String filename,byte[] data){
		try{
			File file = new File(path+filename);
			if(!file.exists()){
				file.createNewFile();
			}
			FileOutputStream fos = new FileOutputStream(file);
			BufferedOutputStream bos = new BufferedOutputStream(fos);
			bos.write(data);
			bos.close();
			fos.close();
		}catch(Exception e){
			e.printStackTrace();
		}
	}

}

```

3、接着自定义编写一个MANIFEST.MF文件，文件目录src/main/resources/META-INF/MANIFEST.MF，内容如下

MANIFEST.MF

```markdown
Manifest-Version: 1.0
Premain-Class: com.hode.ClazzGenerateAgent
Archiver-Version: Plexus Archiver
Built-By: hode
Created-By: Apache Maven 3.3.3
Build-Jdk: 1.8.0_66

```

4、执行命令，完成安装

```markdown
mvn clean install
```

安装完成后，可以在maven仓库目录查看jar包，如目录在d:\repository\com\hode\clazzagent\0.0.1-SNAPSHOT\clazzagent-0.0.1-SNAPSHOT.jar

5、编写一个测试类并运行
<font color="red">注意：运行时需加上jvm参数-javaagent:"D:\repository\com\hode\clazzagent\0.0.1-SNAPSHOT\clazzagent-0.0.1-SNAPSHOT.jar"</font>

```
package com.hode;

public class Test {

	public static void main(String[] args) {
		System.out.println("success");
	}

}
```

在eclipse中在VM arguments中填写，在命令行模式下执行，则如下

```markdown
java -javaagent:"D:\repository\com\hode\clazzagent\0.0.1-SNAPSHOT\clazzagent-0.0.1-SNAPSHOT.jar" com.hode.Test
```


执行完成后，即可以类ClazzGenerateAgent定义的保存目录c:\clazz中查看到Test.class类文件，使用反编译工具可查看具体内容。


<h4><a href="/rar/class-agent.rar"><b>Demo代码下载</b></a></h4>


结束。
