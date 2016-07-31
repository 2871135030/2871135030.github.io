---
layout: post
title: Redis集群安装
date:   2016-04-27 10:33:44
categories: [redis,tools]
---

## Redis集群安装

前文<a href="/tools/redis/2016/04/12/redis-install.html">Redis安装使用</a>介绍过redis的单机版安装，

本文将介绍redis集群安装，本文以redis新的稳定版本redis-3.2.2.tar.gz为例。

1、先按前文步骤安装好单机版redis，确认安装成功后继续后面步骤。下列简要列出步骤

```markdown
tar -xzvf redis-3.2.2.tar.gz

解压完成后进入目录 
cd /software/redis-3.2.2/src

make install

若出现错误gcc：命令未找到，先执行 yum -y install gcc

若出现错误：jemalloc/jemalloc.h：没有那个文件或目录，执行 make MALLOC=libc 完成安装

执行make install后，test一下
make test

若出现You need tcl 8.5 or newer in order to run the Redis test
则执行命令
yum -y install tcl

安装完成后
设置密码
vi /software/redis-3.2.2/redis.conf 为redis设置密码 项 requirepass redis2016
为了方便不同的外部客户端连接，还需暂时屏蔽bind 127.0.0.1这个配置项
cd /software/redis-3.2.2/
mkdir /usr/redis
cd src
cp redis-server /usr/redis
cp redis-benchmark /usr/redis
cp redis-cli /usr/redis
cp ../redis.conf /usr/redis
cd /usr/redis
ps -ef | grep 'redis-server' | grep -v grep| awk '{print $2}'| xargs kill -9
/usr/redis/redis-server /usr/redis/redis.conf & 后台启动运行

使用客户端命令redis-cli连接，进行测试
redis-cli -h 127.0.0.1 -p 6379  -a redis2016

使用以下shell命令可关闭redis服务
ps -ef | grep 'redis-server' | grep -v grep| awk '{print $2}'| xargs kill -9

``` 

到此redis单机版已经安装成功，下面开始配置集群。

```markdown
创建redis集群目录
mkdir /usr/redis-cluster
cd /usr/redis-cluster

分别将文件依次拷贝到指定目录中，这样在准备集群的目录中有四个文件
cp /usr/redis/redis-server /usr/redis-cluster
cp /usr/redis/redis-benchmark /usr/redis-cluster
cp /usr/redis/redis-cli /usr/redis-cluster
cp /usr/redis/redis.conf /usr/redis-cluster

下面编辑redis.conf文件(注意：本例集群在同一台机器上，需注意端口、配置文件路径等)

vi redis.conf
分别找到以下项，并修改对应值，若为注释项则解除注释(需屏蔽密码项#requirepass redis2016，创建集群时不需要密码)
port 7000
daemonize yes
pidfile /var/run/redis7000.pid
cluster-enabled yes
appendonly yes
cluster-node-timeout 15000
cluster-config-file nodes-7000.conf

cp /usr/redis-cluster/redis.conf /usr/redis-cluster/redis-7000.conf

依次编辑redis-7001.conf、redis-7002.conf、redis-7003.conf、redis-7004.conf、redis-7005.conf文件，
注意配置文件中的 7000改为对应值


下面启动集群redis
cd /usr/redis-cluster/
/usr/redis-cluster/redis-server redis-7000.conf &
/usr/redis-cluster/redis-server redis-7001.conf &
/usr/redis-cluster/redis-server redis-7002.conf &
/usr/redis-cluster/redis-server redis-7003.conf &
/usr/redis-cluster/redis-server redis-7004.conf &
/usr/redis-cluster/redis-server redis-7005.conf &

使用以下命令检查是否正常
ps -ef|grep redis

例如：
root      1830     1  0 20:24 ?        00:00:00 /usr/redis-cluster/redis-server *:7000 [cluster]
root      1832     1  0 20:24 ?        00:00:00 /usr/redis-cluster/redis-server *:7001 [cluster]
root      1834     1  0 20:24 ?        00:00:00 /usr/redis-cluster/redis-server *:7002 [cluster]
root      1835     1  0 20:24 ?        00:00:00 /usr/redis-cluster/redis-server *:7003 [cluster]
root      1836     1  0 20:24 ?        00:00:00 /usr/redis-cluster/redis-server *:7004 [cluster]
root      1837     1  0 20:24 ?        00:00:00 /usr/redis-cluster/redis-server *:7005 [cluster]

说明redis各机器正常

此时可以使用命令查看节点
例如：
[root@localhost redis-cluster]# redis-cli -h 127.0.0.1 -p 7000 cluster nodes
ed309eb3944df6eb721664625525281a0e93bca0 :7000 myself,master - 0 0 0 connected
[root@localhost redis-cluster]# redis-cli -h 127.0.0.1 -p 7001 cluster nodes
46d3a1772aa23ec160fa9b3a789451d855417cc8 :7001 myself,master - 0 0 0 connected
[root@localhost redis-cluster]# redis-cli -h 127.0.0.1 -p 7002 cluster nodes
77404e0a3c32fa80785b06ece1095a162f6fed7b :7002 myself,master - 0 0 0 connected
[root@localhost redis-cluster]# redis-cli -h 127.0.0.1 -p 7003 cluster nodes
1bd8506192c44ec87f599bef99310b9c55c1b1a4 :7003 myself,master - 0 0 0 connected
[root@localhost redis-cluster]# redis-cli -h 127.0.0.1 -p 7004 cluster nodes
a23730e476a3fdf178728a3a57f662c84385019c :7004 myself,master - 0 0 0 connected
[root@localhost redis-cluster]# redis-cli -h 127.0.0.1 -p 7005 cluster nodes
5827cb5db9a9bac200ce5b3814faa6a0cfbb20f3 :7005 myself,master - 0 0 0 connected

可以看到每一个redis服务都是master节点，接下来要执行redis创建集群命令来完成创建集群

创建集群命令为ruby命令，需先安装ruby命令
yum -y install ruby
yum -y install rubygems

gem install redis(这里若因为无法连接gem服务器,可手工下载并安装，手工下载地址
wget https://rubygems.global.ssl.fastly.net/gems/redis-3.3.1.gem ，下载完成后执行gem install -l ./redis-3.3.1.gem完成安装)


安装完成后创建集群

cp /software/redis-3.2.2/src/redis-trib.rb /usr/redis-cluster

/usr/redis-cluster/redis-trib.rb  create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005

输入yes完成集群配置

示例如下：
[root@localhost redis-cluster]# /usr/redis-cluster/redis-trib.rb  create --replicas 1 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
127.0.0.1:7000
127.0.0.1:7001
127.0.0.1:7002
Adding replica 127.0.0.1:7003 to 127.0.0.1:7000
Adding replica 127.0.0.1:7004 to 127.0.0.1:7001
Adding replica 127.0.0.1:7005 to 127.0.0.1:7002
M: ed309eb3944df6eb721664625525281a0e93bca0 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 46d3a1772aa23ec160fa9b3a789451d855417cc8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: 77404e0a3c32fa80785b06ece1095a162f6fed7b 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
S: 1bd8506192c44ec87f599bef99310b9c55c1b1a4 127.0.0.1:7003
   replicates ed309eb3944df6eb721664625525281a0e93bca0
S: a23730e476a3fdf178728a3a57f662c84385019c 127.0.0.1:7004
   replicates 46d3a1772aa23ec160fa9b3a789451d855417cc8
S: 5827cb5db9a9bac200ce5b3814faa6a0cfbb20f3 127.0.0.1:7005
   replicates 77404e0a3c32fa80785b06ece1095a162f6fed7b
Can I set the above configuration? (type 'yes' to accept): yes  
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join.....
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: ed309eb3944df6eb721664625525281a0e93bca0 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
M: 46d3a1772aa23ec160fa9b3a789451d855417cc8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
M: 77404e0a3c32fa80785b06ece1095a162f6fed7b 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
M: 1bd8506192c44ec87f599bef99310b9c55c1b1a4 127.0.0.1:7003
   slots: (0 slots) master
   replicates ed309eb3944df6eb721664625525281a0e93bca0
M: a23730e476a3fdf178728a3a57f662c84385019c 127.0.0.1:7004
   slots: (0 slots) master
   replicates 46d3a1772aa23ec160fa9b3a789451d855417cc8
M: 5827cb5db9a9bac200ce5b3814faa6a0cfbb20f3 127.0.0.1:7005
   slots: (0 slots) master
   replicates 77404e0a3c32fa80785b06ece1095a162f6fed7b
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@localhost redis-cluster]# 

使用以下命令检查集群
/usr/redis-cluster/redis-trib.rb check 127.0.0.1:7000

示例检查结果
[root@localhost redis-cluster]# /usr/redis-cluster/redis-trib.rb check
[ERR] Wrong number of arguments for specified sub command
[root@localhost redis-cluster]# /usr/redis-cluster/redis-trib.rb check 127.0.0.1:7000
>>> Performing Cluster Check (using node 127.0.0.1:7000)
M: ed309eb3944df6eb721664625525281a0e93bca0 127.0.0.1:7000
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 77404e0a3c32fa80785b06ece1095a162f6fed7b 127.0.0.1:7002
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: a23730e476a3fdf178728a3a57f662c84385019c 127.0.0.1:7004
   slots: (0 slots) slave
   replicates 46d3a1772aa23ec160fa9b3a789451d855417cc8
M: 46d3a1772aa23ec160fa9b3a789451d855417cc8 127.0.0.1:7001
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 5827cb5db9a9bac200ce5b3814faa6a0cfbb20f3 127.0.0.1:7005
   slots: (0 slots) slave
   replicates 77404e0a3c32fa80785b06ece1095a162f6fed7b
S: 1bd8506192c44ec87f599bef99310b9c55c1b1a4 127.0.0.1:7003
   slots: (0 slots) slave
   replicates ed309eb3944df6eb721664625525281a0e93bca0
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
[root@localhost redis-cluster]# 

```


结束。
