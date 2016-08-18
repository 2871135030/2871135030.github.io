---
layout: post
title: Maven命令:deploy和release命令
date:   2016-05-09 12:08:48
categories: [maven,tools]
---

## Maven命令:deploy和release命令


本文介绍deploy及release命令，首先需在maven配置文件conf/settings.xml配置好私服

示例settings.xml

```markdown
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

<pluginGroups>

</pluginGroups>

<proxies>

</proxies>

<servers>
		<!-- 配置发布私服用户名密码 -->
		<server>
			<id>release</id>
			<username>admin</username>
			<password>admin123</password>
		</server>
		<server>
			<id>snapshots</id>
			<username>admin</username>
			<password>admin123</password>
		</server>
</servers>

<!--设置私库mirror 表示maven所有的请求都由nexus来处理-->
<mirrors>
	<mirror> 
        <id>nexus</id> 
        <mirrorOf>*</mirrorOf> 
        <name>Nexus Mirror.</name> 
        <url>http://192.168.245.130:8081/nexus/content/groups/public</url> 
    </mirror>
</mirrors>


<profiles>
	<!-- 仓库及插件仓库 -->
	<profile>
		<id>nexus</id>
		<repositories>
			<repository>
				<id>nexus</id>
				<name>Nexus</name>
				<url>http://192.168.245.130:8081/nexus/content/groups/public/</url>
				<release><enabled>true</enabled></release>
				<snapshots><enabled>true></enabled></snapshots>
			</repository>
		</repositories>
		<pluginRepositories>
			<pluginRepository>
				<id>nexus</id>
				<name>Nexus</name>
				<url>http://192.168.245.130:8081/nexus/content/groups/public/</url>
				<release><enabled>true</enabled></release>
				<snapshots><enabled>true></enabled></snapshots>
			</pluginRepository>
		</pluginRepositories>
	</profile>

</profiles>
	<!-- 激活 -->
	<activeProfiles>
		<activeProfile>nexus</activeProfile>
	</activeProfiles>
	
</settings>

```

同时项目的pom.xml需要配置私服

```markdown
	<!-- 配置代码库 -->
	<scm>
		<connection>scm:svn:http://bbs.bao.org/svn/allhode/jaxrs/jaxrs-interface/</connection>
		<developerConnection>scm:svn:http://bbs.bao.org/svn/allhode/jaxrs/jaxrs-interface/</developerConnection>
	</scm>

	<build>
		<!-- 此插件配置release版本发布之后打tag位置 -->
		<plugins>
			<plugin>
			  <groupId>org.apache.maven.plugins</groupId>
			  <artifactId>maven-release-plugin</artifactId>
			  <version>2.0-beta-7</version>
			  <configuration>
				<tagBase>http://bbs.bao.org/svn/allhode/jaxrs/tags</tagBase>
			  </configuration>
			</plugin>
		</plugins>
	</build>
	
	<!--发布私服地址-->
	<distributionManagement>  
        <repository>  
            <id>release</id>  
            <name>Project Release</name>  
            <url>http://192.168.245.130:8081/nexus/content/repositories/releases/</url>  
        </repository>
		<snapshotRepository>  
            <id>snapshots</id>  
            <name>Project SNAPSHOTS</name>  
            <url>http://192.168.245.130:8081/nexus/content/repositories/snapshots/</url>  
        </snapshotRepository>
	</distributionManagement>
```

执行命令mvn clean deploy将项目发布到私服中（详细的说明可参考maven官方文档）

<h6><font color="red">注意：deploy会根据pom.xml中配置的版本是否为快照版本发布到snapshots或release。
如&lt;version&gt;0.0.4-SNAPSHOT&lt;/version&gt; 其中版本号含SNAPSHOT字样，将发布到snapshots快照库，否则发布到release发行库。发布的地址即为项目
pom.xml中配置的发布私服地址。</font></h6>


下面分析下mvn release:prepare以及mvn release:perform

在项目开发的过程中，经过开发测试已有一个稳定的版本时，我们就需要发布一个release正式版本，而不是snapshots快照版本。手工完成此过程大概如下：
（若此时版本为0.0.4-SNAPSHOT）

```markdown
1、修改版本为0.0.4正式版本，提交
2、执行mvn clean deploy发布将0.0.4版本发布到release发行库
3、将此0.0.4正式版本源码在svn上打一个tag
4、升级版本为0.0.5-SNAPSHOT，提交
```

如果项目数量不多或需要升级的依赖不多，手工操作也无妨；但若项目数量众多，依赖繁杂的话，就要借助工具帮忙简化发布流程了。

```markdown
mvn release:prepare
此命令，需在pom.xml中配置scm代码库，配置maven-release-plugin插件自动打tag，执行此命令时会提示是否按默认的版本发布方式进行，
若需要变更版本规则（如升级次版本号），执行完成后项目目录下会生成pom.xml.releaseBackup及release.properties两个文件，其中有对应发布描述。

mvn release:perform
此命令完成最终发布

```

<font color="red">注意：mvn release:prepare 与 mvn release:perform 为一对命令</font>
mvn release:prepare主要处理流程为将0.0.4-SNAPSHOT变更为0.0.4提交到版本库，再打个tag，完成后升级版本为0.0.5-SNAPSHOT。

mvn release:perform主要处理流程为根据上一步生成的配置release.properties从tag中获取源码checkout到target/checkout目录中，完成发布。



结束。
