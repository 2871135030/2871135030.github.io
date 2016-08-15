---
layout: post
title: 汇总一些linux安装java应用环境相关操作命令
date:   2016-04-14 15:32:57
categories: java
---

## 汇总一些linux安装java应用环境相关操作命令

收集汇总安装 CentOS-6.7-x86_64-minimal.iso 后安装java运行环境的一些命令操作。

*****************
1、安装CentOS-6.7-x86_64-minimal，在vmware中需先创建空白机器，然后导入iso安装盘进行安装，若直接安装将提示安装失败。

*****************
2、一般设置直接连接物理网络，可大幅减轻网络连接占用的系统资源，配置成自动获取ip如下
{% highlight ruby %}
编辑网卡配置文件，命令： vim /etc/sysconfig/network-scripts/ifcfg-eth0
更改ONBOOT=yes
更改BOOTPROTO为：
BOOTPROTO=dhcp
其他的如IPADDR、NETMASK等#号注释掉，保存退出
重启网络连接，命令： service network restart
{% endhighlight %}
*****************
3、安装 sz rz上传下载工具
{% highlight ruby %}
运行：yum list | grep “lrzsz”查看lrzsz包
运行：sudo yum install lrzsz.x86_64 安装
{% endhighlight %}
	
*****************
4、安装jdk
{% highlight ruby %}
下载安装包jdk-7u67-linux-x64.tar.gz
命令：tar -xzvf jdk-7u67-linux-x64.tar.gz
设置环境变量：vi /etc/profile 在最后加入以入几行
export JAVA_HOME=/software/jdk1.7.0_67
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH
输入命令 source /etc/profile 使环境变量立即生效
命令 java -version 查看jdk是否安装成功
{% endhighlight %}
*****************
5、安装tomcat
{% highlight ruby %}
下载安装包apache-tomcat-7.0.68.tar.gz
命令：tar -xzvf apache-tomcat-7.0.68.tar.gz
启动tomcat命令 
/software/apache-tomcat-7.0.68/bin/startup.sh
tail -f /software/apache-tomcat-7.0.68/logs/*out
快速关闭tomcat命令 
ps -ef | grep '/software/apache-tomcat-7.0.68' | grep -v grep| awk '{print $2}'| xargs kill -9
{% endhighlight %}
6、防火墙
{% highlight ruby %}
永久打开或则关闭
chkconfig iptables on 
chkconfig iptables off 
即时生效：重启后还原 
service iptables start 
service iptables stop 
{% endhighlight %}
*****************
7、安装jenkins
{% highlight ruby %}
下载jenkins.war包，将war包放入tomcat中启动tomcat
<Context path="" docBase="/software/tomcat-jenkins/jenkins.war" />
ps -ef | grep '/software/tomcat-jenkins' | grep -v grep| awk '{print $2}'| xargs kill -9
rm -rf /software/tomcat-jenkins/logs/*
rm -rf /software/tomcat-jenkins/webapps/*
rm -rf /software/tomcat-jenkins/work/*
/software/tomcat-jenkins/bin/startup.sh
tail -f /software/tomcat-jenkins/logs/*out
安装完成后，进入jenkins->系统管理->Configure Global Security设置
{% endhighlight %}
*****************
8、安装zip unzip
{% highlight ruby %}
yum install zip unzip  
例如：unzip project.war -d project
{% endhighlight %}
*****************
9、安装maven
{% highlight ruby %}
下载apache-maven-3.3.1-bin.zip
执行命令 unzip apache-maven-3.3.1-bin.zip
设置环境变量：vi /etc/profile 在最后加入以入几行
export PATH=$JAVA_HOME/bin:$PATH:/software/apache-maven-3.3.1/bin
输入命令 source /etc/profile 使环境变量立即生效
{% endhighlight %}
*****************
10、在线安装svn客户端
{% highlight ruby %}
yum -y install subversion
{% endhighlight %}
*****************	
11、使用svn检出项目/更新项目
{% highlight ruby %}
cd /project
svn checkout http://192.168.1.100/svn/hode/jaxws-server
输入用户名及密码即可
cd jaxws-server
svn up 即可更新
若svn地址变更过，需要relocate
svn switch --relocate http://192.168.1.100/svn/hode/jaxws-server http://192.168.88.101/svn/hode/jaxws-server
svn checkout http://192.168.88.101/svn/hode/jaxws-client
svn checkout http://192.168.88.101/svn/hode/jaxws-interface
{% endhighlight %}
*****************	
12、安装zookeeper
{% highlight ruby %}
下载 zookeeper-3.4.8.tar.gz 包
tar -xzvf zookeeper-3.4.8.tar.gz
进入zookeeper目录下的conf子目录, cp -rf zoo_sample.cfg  zoo.cfg 复制一份配置文件
vi zoo.cfg完成配置(其它默认便可)
dataDir=/data/zookeeper
dataLogDir=/data/zookeeper/lo
单机模式已配置好，启动命令为  bin/zkServer.sh start 
Server启动之后, 就可以启动client连接server了, 执行脚本
bin/zkCli.sh -server localhost:2181  便可连接成功
ls /可查看相关节点
{% endhighlight %}	
*****************	
13、安装redis
{% highlight ruby %}
下载redis-2.8.19.tar.gz
tar -xzvf redis-2.8.19.tar.gz
进入目录 /software/redis-2.8.19
make 完成安装
若出现错误gcc：命令未找到，先执行 yum -y install gcc
若出现错误：jemalloc/jemalloc.h：没有那个文件或目录 
执行 make MALLOC=libc 完成安装
vi /software/redis-2.8.19/redis.conf 为redis设置密码 项 requirepass redis2016
mkdir /usr/redis
cp redis-server /usr/redis
cp redis-benchmark /usr/redis
cp redis-cli /usr/redis
cp ../redis.conf /usr/redis
cd /usr/redis
ps -ef | grep 'redis-server' | grep -v grep| awk '{print $2}'| xargs kill -9
./redis-server & 后台运行
{% endhighlight %}
*****************	
14、安装Git
{% highlight ruby %}
执行命令 yum -y install git 完成安装
{% endhighlight %}
*****************	
15、安装telnet
{% highlight ruby %}
执行命令 yum -y install telnet 完成安装
{% endhighlight %}

结束。
