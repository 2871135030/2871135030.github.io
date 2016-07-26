---
layout: post
title: Zookeeper集群环境安装
date:   2016-04-20 13:03:13
categories: java
---

## Zookeeper集群环境安装

ZooKeeper是一个分布式开源框架，提供了协调分布式应用的基本服务，它向外部应用暴露一组通用服务——分布式同步（Distributed Synchronization）、命名服务（Naming Service）、集群维护（Group Maintenance）等，简化分布式应用协调及其管理的难度，提供高性能的分布式服务。ZooKeeper本身可以以Standalone模式安装运行，不过它的长处在于通过分布式ZooKeeper集群（一个Leader，多个Follower），基于一定的策略来保证ZooKeeper集群的稳定性和可用性，从而实现分布式应用的可靠性。

单机安装Zookeeper已在文中<a href="http://www.mixfate.com/tools/2016/04/11/zookeeper-install.html">ZooKeeper安装使用</a>介绍过，在实际应用中我们将zookeeper集群部署，构建高可用的服务，避免单点故障。

1、在目录/software/zookeeper-cluster/中解压三份zookeeper安装包（zookeeper-3.4.8.tar.gz包），分别为zookeeper-3.4.8-node1、zookeeper-3.4.8-node2、zookeeper-3.4.8-node3，
集群中所有的结点作为一个整体对分布式应用提供服务，本文中在同一台机器上安装3个
分别编辑note1、note2、note3节点的配置文件，文件目录conf/zoo.cfg
{% highlight ruby %}
tickTime=2000
initLimit=10
syncLimit=5
dataDir=/software/zookeeper-cluster/zookeeper-3.4.8-node1/data
clientPort=2181
server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889
{% endhighlight %}

<font color="red">注意：dataDir、clientPort两个参数三个节点不同</font>
dataDir顾名思义就是Zookeeper保存数据的目录,默认情况下Zookeeper将写数据的日志文件也保存在这个目录里，三个节点各自保存在节点的data目录内；

clientPort这个端口就是客户端连接Zookeeper服务器的端口,Zookeeper会监听这个端口接受客户端的访问请求；

server.1=localhost:2887:3887
server.2=localhost:2888:3888
server.3=localhost:2889:3889

server.A=B：C：D

A是一个数字,表示这个是第几号服务器,B是这个服务器的ip地址

C第一个端口用来集群成员的信息交换,表示的是这个服务器与集群中的Leader服务器交换信息的端口

D是在leader挂掉时专门用来进行选举leader所用

2、创建ServerID标识

集群模式下还要配置一个文件myid,这个文件在dataDir目录下,这个文件里面就有一个数据就是A的值,在上面配置文件中zoo.cfg中配置的dataDir路径中创建myid文件(不要后缀)，文件内容对应server.A中的A值，
即三个节点中的myid内容分别为1,2,3，cat查看

{% highlight ruby %}
[root@localhost zookeeper-cluster]# cat /software/zookeeper-cluster/zookeeper-3.4.8-node1/data/myid
1
[root@localhost zookeeper-cluster]# cat /software/zookeeper-cluster/zookeeper-3.4.8-node2/data/myid
2
[root@localhost zookeeper-cluster]# cat /software/zookeeper-cluster/zookeeper-3.4.8-node3/data/myid
3
{% endhighlight %}

3、启动和停止
启动命令如下
{% highlight ruby %}
cd /software/zookeeper-cluster/

./zookeeper-3.4.8-node1/bin/zkServer.sh start
./zookeeper-3.4.8-node2/bin/zkServer.sh start
./zookeeper-3.4.8-node3/bin/zkServer.sh start

{% endhighlight %}
启动完成后，可以使用以下命令查看各节点情况
{% highlight ruby %}
echo 查看启动情况
echo stat|nc localhost 2181
echo stat|nc localhost 2182
echo stat|nc localhost 2183
{% endhighlight %}

示例结果如下（可以看到节点1、2为follower，节点3为leader）
{% highlight ruby %}
[root@localhost zookeeper-cluster]# echo 查看启动情况
查看启动情况
[root@localhost zookeeper-cluster]# echo stat|nc localhost 2181
Zookeeper version: 3.4.8--1, built on 02/06/2016 03:18 GMT
Clients:
 /0:0:0:0:0:0:0:1:59347[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x2200000032
Mode: follower
Node count: 1653
echo stat|nc localhost 2182
[root@localhost zookeeper-cluster]# echo stat|nc localhost 2182
echo stat|nc localhost 2183
Zookeeper version: 3.4.8--1, built on 02/06/2016 03:18 GMT
Clients:
 /0:0:0:0:0:0:0:1:51783[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x2200000032
Mode: follower
Node count: 1653
[root@localhost zookeeper-cluster]# echo stat|nc localhost 2183
Zookeeper version: 3.4.8--1, built on 02/06/2016 03:18 GMT
Clients:
 /0:0:0:0:0:0:0:1:45005[0](queued=0,recved=1,sent=0)

Latency min/avg/max: 0/0/0
Received: 2
Sent: 1
Connections: 1
Outstanding: 0
Zxid: 0x2600000000
Mode: leader
Node count: 1653
{% endhighlight %}

若需要关闭，可以使用stop命令停止
{% highlight ruby %}
cd /software/zookeeper-cluster/
./zookeeper-3.4.8-node1/bin/zkServer.sh stop
./zookeeper-3.4.8-node2/bin/zkServer.sh stop
./zookeeper-3.4.8-node3/bin/zkServer.sh stop
{% endhighlight %}

结束。
