---
layout: post
title: Gradle安装与简单使用介绍
date:   2016-05-18 12:00:58
categories: [tools]
---

## Gradle安装与简单使用介绍

Gradle是一种构建工具，它抛弃了基于XML的构建脚本，取而代之的是采用一种基于Groovy的内部领域特定语言。详细介绍在此就不作过多描述了，直接以例子切入。

1、要使用Gradle，首先是安装环境，本文演示在linux(centos6.5)中安装使用。

```markdown
打开gradle官方下载地址 https://gradle.org/gradle-download/ （不保证此网址一直不变更），可以看到官方给出两句命令完成Gradle的安装，如下（第2、3步使用）：

$ curl -s https://get.sdkman.io | bash
$ sdk install gradle 3.1

```

2、安装sdk工具

```markdown
需先安装unzip工具执行命令
yum install zip unzip -y
安装sdkman
curl -s https://get.sdkman.io | bash
安装完成后按提示执行以下命令
source "/root/.sdkman/bin/sdkman-init.sh"
使用sdk help命令检查安装结果
```

3、安装gradle

```markdown
执行以下命令，完成gradle安装（在线安装耗时相对较长）
sdk install gradle 3.1
安装结束后可使用以下命令检查安装情况
gradle -v
```

4、简单创建一个java工程，或java web工程，注意目录结构与maven标准目录结构一致

目录结构如下(若非web项目，则无目录webapp)：

```markdown
src
 ├── main
 │   ├── java
 │   ├── resources
 │   └── webapp
 └── test
     ├── java
     └── resources
```

创建此目录结构可直接使用shell命令（建议采用），或使用gradle的task来初始化目录，下面分别介绍

5、使用shell命令

```markdown
非web项目
mkdir -p src/{main,test}/{java,resources}
web项目
mkdir -p src/{main,test}/{java,resources} src/main/webapp
创建完成后可使用tree命令查看目录结构，若未安装tree命令，可执行yum install tree -y完成安装
```

6、使用gradle任务初始化，在根目录下新建一个文件build.gradle(类似maven的pom.xml)，内容分别如下

非web项目build.gradle如下

```markdown
apply plugin:'java'
apply plugin:'eclipse'

task createJavaProject <<{
sourceSets*.java.srcDirs*.each{it.mkdirs()}
sourceSets*.resources.srcDirs*.each{it.mkdirs()}
}
```

建立目录时执行命令 gradle createJavaProject eclipse，执行完成后同样可使用tree命令查看目录结构

web项目build.gradle如下

```markdown
apply plugin:'java'
apply plugin:'eclipse-wtp'
apply plugin:'war'

task createJavaProject <<{
sourceSets*.java.srcDirs*.each{it.mkdirs()}
sourceSets*.resources.srcDirs*.each{it.mkdirs()}
}

task createWebProject(dependsOn:'createJavaProject') <<{
def webAppDir=file("$webAppDirName")
webAppDir.mkdirs()
}
```

建立目录时执行命令 gradle createWebProject eclipse，执行完成后同样可使用tree命令查看目录结构

至此，目录简单介绍完毕。


7、下面简单写一个Test类，使用gradle完成编译打包

先创建一个java工程，编写build.gradle，同时注意增加了内容jar且指定了Main-Class

build.gradle

```markdown
apply plugin:'java'
apply plugin:'eclipse'

task createJavaProject <<{
sourceSets*.java.srcDirs*.each{it.mkdirs()}
sourceSets*.resources.srcDirs*.each{it.mkdirs()}
}
jar {
 manifest{
  attributes 'Main-Class':'com.hode.Test'
 }
}
```

执行命令 gradle createJavaProject eclipse 完成目录初始化

创建Test.java类（实际目录src/main/java/com/hode），内容如下

```markdown
package com.hode;

public class Test{
	public static void main(String[] args){
		System.out.println("this is a test class for gradle");
	}
}
```

执行命令gradle build完成构建，可以看到tree目录结构如下，build为编译结果

```markdown
.
├── build
│   ├── classes
│   │   └── main
│   │       └── com
│   │           └── hode
│   │               └── Test.class
│   ├── dependency-cache
│   ├── libs
│   │   └── javaProject.jar
│   └── tmp
│       ├── compileJava
│       └── jar
│           └── MANIFEST.MF
├── build.gradle
└── src
    ├── main
    │   ├── java
    │   │   └── com
    │   │       └── hode
    │   │           └── Test.java
    │   └── resources
    └── test
        ├── java
        └── resources
```

最后使用java -jar build/libs/javaProject.jar 查看测试类的结果。

下一篇简单简单介绍一下构建一个spring项目。

完毕。

