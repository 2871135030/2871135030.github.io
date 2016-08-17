---
layout: post
title: Nexus2.9.0 私服 linux安装使用
date:   2016-05-08 11:22:48
categories: [maven,tools]
---

## Nexus2.9.0 私服 linux安装使用

```markdown
下载nexus-2.9.0-bundle.zip，使用解压命令

unzip nexus-latest-bundle.zip -d . 

解压到目录/software/nexus-2.7.2-03

若为root用户需在 /etc/profile最后添加一行export RUN_AS_USER=root 并source /etc/profile使其生效

cd /software/nexus-2.7.2-03/bin

执行
./nexus start

若存在Caused by: java.net.UnknownHostException: centos-b 异常，则直接修改hosts文件(vi /etc/hosts)，将原127.0.0.1替换成127.0.0.1 localhost centos-b即可

启动成功后
http://192.168.88.102:8081/nexus/index.html 即可打开界面

新搭建的neuxs环境只是一个空的仓库，需要手动和远程中心库进行同步，nexus默认是关闭远程索引下载，最重要的一件事情就是开启远程索引下载。登陆nexus系统，默认用户名密码为admin/admin123。
点击左边Administration菜单下面的Repositories，找到右边仓库列表中的三个仓库Apache Snapshots，Codehaus Snapshots和Maven Central，然后在每个仓库的configuration下把Download Remote Indexes修改为true。

```

maven的安装配置settings.xml配置修改如下
<profiles>标签内增加以下代码，表示指定仓库及插件仓库

```markdown
	<profile>
		<id>nexus</id>
		<repositories>
			<repository>
				<id>nexus</id>
				<name>Nexus</name>
				<url>http://192.168.88.102:8081/nexus/content/groups/public/</url>
				<release><enabled>true</enabled></release>
				<snapshots><enabled>true></enabled></snapshots>
			</repository>
		</repositories>
		<pluginRepositories>
			<pluginRepository>
				<id>nexus</id>
				<name>Nexus</name>
				<url>http://192.168.88.102:8081/nexus/content/groups/public/</url>
				<release><enabled>true</enabled></release>
				<snapshots><enabled>true></enabled></snapshots>
			</pluginRepository>
		</pluginRepositories>
	</profile>
```

<settings>标签内增加以下代码，表示激活

```markdown
	<activeProfiles>
		<activeProfile>nexus</activeProfile>
	</activeProfiles>
```

<settings>标签内增加以下代码，表示发布

```markdown
	<distributionManagement>  
        <repository>  
            <id>release</id>  
            <name>User Project Release</name>  
            <url>http://192.168.88.102:8081/nexus/content/repositories/releases/</url>  
        </repository>  
  
        <snapshotRepository>  
            <id>snapshots</id>  
            <name>User Project SNAPSHOTS</name>  
            <url>http://192.168.88.102:8081/nexus/content/repositories/snapshots/</url>  
        </snapshotRepository>  
    </distributionManagement>
```

并且在<settings>标签下的<servers>标签中添加

```markdown
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
```

同时pom.xml中也需要添加

```markdown
	<distributionManagement>  
        <repository>  
            <id>release</id>  
            <name>Project Release</name>  
            <url>http://192.168.88.102:8081/nexus/content/repositories/releases/</url>  
        </repository>
		<snapshotRepository>  
            <id>snapshots</id>  
            <name>Project SNAPSHOTS</name>  
            <url>http://192.168.88.102:8081/nexus/content/repositories/snapshots/</url>  
        </snapshotRepository>
	</distributionManagement>
```

执行命令mvn deploy便可


结束。
