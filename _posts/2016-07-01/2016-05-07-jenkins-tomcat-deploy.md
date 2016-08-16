---
layout: post
title: jenkins远程发布应用到tomcat中，使用shell脚本或jenkins deploy插件
date:   2016-05-07 12:11:48
categories: [tools]
---

## jenkins远程发布应用到tomcat中，使用shell脚本或jenkins deploy插件

Jenkins将构建完成后的war包部署到tomcat有很多方式可操作，本文介绍两种操作，
分别是使用jenkins的Deploy to container Plugin插件及执行linux ssh方式。

1、使用插件方式，插件为Deploy to container Plugin，需先安装好插件。

配置远程机器的tomcat(本例使用tomcat-7.0.68)，因此种方式需使用tomcat管理发布，需首先配置用户

进入tomcat根目录

```markdown
vi conf/tomcat-users.xml  在标签</tomcat-users>前添加如下代码
  <role rolename="tomcat"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <user username="tomcat" password="tomcat" roles="tomcat,manager-gui,manager-script,manager-jmx,manager-status"/>
  
```

即授权用户名 tomcat,密码tomcat可操作tomcat管理控制台。

启动tomcat，接下来由jenkins中配置

打开Jenkins的对应需要配置的项目，打开配置，找到“构建后操作”，“增加构建后操作步骤”中选择
Deploy war/ear to a container，打开后Add Container中选择Tomcat 7.x

配置样例如下（更改为实际项目即可）

```markdown

WAR/EAR files:target/jaxrs-server-0.0.1-SNAPSHOT.war

Context path:jaxrs

Containers
  Manager user name:tomcat
  Manager password:tomcat
  Tomcat URL:http://192.168.88.103:8090/

```

保存，完成构建即可。这种方式需先保证目录tomcat已经启动方可正常发布（个人建议采用第二种方式，以开发环境为例
tomcat启动后经过反复的删除部署，重新部署，应用的使用过程，可能内存的清理不是非常干净，导致部署不成功，需手工重启）。

2、使用shell脚本部署

所需配置前提：jenkins部署的机器，可直接操作scp ssh远程tomcat机器

在项目的配置中，“构建”->“增加构建步骤” 中选择Execute shell，当然可创建多个

先创建一个Execute shell其中的Command如下，目的是拷贝war包到目标机器上，这一句比较简单只完成拷贝功能

```markdown
scp -p /project/jenkins_space/workspace/jaxrs-server/target/jaxrs-server-0.0.1-SNAPSHOT.war root@192.168.88.103:/project/jaxrs-server-0.0.1-SNAPSHOT.war
```


再创建一个Execute shell其中的Command如下，其目的是执行目标机器上的shell脚本

```markdown
#!/bin/sh

ssh root@192.168.88.103 '/software/apache-tomcat-7.0.68/kill.sh'

ssh root@192.168.88.103 '/software/apache-tomcat-7.0.68/deploy.sh'

```

在目标机器上创建kill.sh(负责kill tomcat)及deploy.sh，并且chmod修改执行权限。

kill.sh如下

```markdown
#!/bin/bash
ps -ef | grep '/software/apache-tomcat-7.0.68' | grep -v grep| awk '{print $2}'| xargs kill -9
```

deploy.sh如下

```markdown
#!/bin/bash
export JAVA_HOME=/software/jdk1.7.0_67
cd /software/apache-tomcat-7.0.68
rm -rf logs/* work/* webapps/*

cd bin
./startup.sh

cd ..

while true
do
  echo tomcat is running
  str=`grep -e "Server startup in [0-9]\+ ms" logs/*out`

  len=`echo $str|wc -L`

  if [ "$len" -gt 0 ]
    then
      cat logs/*out
      break
    else
      echo please waiting...
  fi

  sleep 5

done
```

说明：/software/apache-tomcat-7.0.68此tomcat配置指向scp目的war包，deploy.sh为启动tomcat，
由于jenkins无法检测到tomcat是否启动成功，deploy.sh添加了一段shell脚本，通过判断服务器的启动日志是否正常。


结束。
