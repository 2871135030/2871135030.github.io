---
layout: post
title: zookeeper安装使用
date:   2016-04-11 05:32:57
categories: tools
---

## ZooKeeper安装使用

下载 zookeeper-3.4.8.tar.gz 包

tar -xzvf zookeeper-3.4.8.tar.gz

进入zookeeper目录下的conf子目录, cp -rf zoo_sample.cfg  zoo.cfg 复制一份配置文件

vi zoo.cfg完成配置(其它默认便可)
{% highlight ruby %}
dataDir=/data/zookeeper
dataLogDir=/data/zookeeper/log
{% endhighlight %}
单机模式已配置好，启动命令为  

bin/zkServer.sh start 

Server启动之后, 就可以启动client连接server了, 执行脚本

bin/zkCli.sh -server localhost:2181  便可连接成功

ls / 命令可查看相关节点
	
	
结束。
