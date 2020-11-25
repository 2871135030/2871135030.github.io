---
layout: post
title: phoenix操作hbase2.0.6
date:   2021-11-12 14:15:06
categories: [mysql]
---

#### phoenix操作hbase2.0.6

##### 1、安装`jdk` `hbase2.0.6` `phoenix`
分别下载对应版本的jdk、hbase2.0.6、phoenix安装包如下，并解压到目录/software中，可从phoenix官网看到版本支持hbase2.0 http://archive.apache.org/dist/phoenix/

```
[root@hbase software]# ll *.gz
-rw-r--r--. 1 root root 436868323 Aug  2 00:00 apache-phoenix-5.0.0-HBase-2.0-bin.tar.gz
-rw-r--r--. 1 root root 132790398 Sep 21 09:52 hbase-2.0.6-bin.tar.gz
-rw-r--r--. 1 root root 189736377 Nov  3  2017 jdk-8u151-linux-x64.tar.gz
[root@hbase software]# 
```

配置环境变量，在/etc/profile文件最后添加以下配置

```
export JAVA_HOME=/software/jdk1.8.0_151
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

export HBASE_HOME=/software/hbase-2.0.6
export PATH=$HBASE_HOME/bin:$PATH


export PHOENIX_HOME=/software/apache-phoenix-5.0.0-HBase-2.0-bin
export PHOENIX_CLASSPATH=$PHOENIX_HOME
export PATH=$PATH:$PHOENIX_HOME/bin
[root@hbase software]#source /etc/profile 使配置生效
```

修改时区并禁用ipv6并关闭防火墙

```
1、timedatectl set-timezone Asia/Shanghai
2、 vi /etc/sysctl.conf 增加配置 net.ipv6.conf.all.disable_ipv6=1
3、vi /etc/sysconfig/network 增加配置 NETWORKING_IPV6=no
4、vi /etc/sysconfig/network-scripts/ifcfg-ens33 将IPV6INIT改为no
5、sysctl -p 使配置生效
```

将phoenix解压目录中的依赖jar包拷贝到hbase的lib目录中

```
[root@hbase apache-phoenix-5.0.0-HBase-2.0-bin]# cp phoenix-5.0.0-HBase-2.0-server.jar phoenix-core-5.0.0-HBase-2.0.jar /software/hbase-2.0.6/lib/
[root@hbase apache-phoenix-5.0.0-HBase-2.0-bin]# 
```

编辑hbase安装目录下conf/hbase-site.xml，configuration标签中增加以下配置

```xml
<property>
    <name>hbase.cluster.distributed</name>
    <value>false</value>
  </property>

<property>
    <name>hbase.rootdir</name>
    <value>file:///data/hbase</value>
  </property>


<property>
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/data/zookeeper</value>
  </property>


  <property>
    <name>hbase.unsafe.stream.capability.enforce</name>
    <value>false</value>
  </property>
```

简单起见，hbase直接单机部署启动，使用命令 `start-hbase.sh`


修改phoenix执行脚本权限

```
chmod 777 psql.py sqlline.py 
```

2、使用phoenix操作hbase

```
[root@hbase conf]#  sqlline.py localhost:2181
Setting property: [incremental, false]
Setting property: [isolation, TRANSACTION_READ_COMMITTED]
issuing: !connect jdbc:phoenix:localhost:2181 none none org.apache.phoenix.jdbc.PhoenixDriver
Connecting to jdbc:phoenix:localhost:2181
20/09/21 23:53:39 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Connected to: Phoenix (version 5.0)
Driver: PhoenixEmbeddedDriver (version 5.0)
Autocommit status: true
Transaction isolation: TRANSACTION_READ_COMMITTED
Building list of tables and columns for tab-completion (set fastconnect to true to skip)...
133/133 (100%) Done
Done
sqlline version 1.2.0
0: jdbc:phoenix:localhost:2181> !tables
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+-+
| TABLE_CAT  | TABLE_SCHEM  | TABLE_NAME  |  TABLE_TYPE   | REMARKS  | TYPE_NAME  | SELF_REFERENCING_COL_NAME  | REF_GENERATION  | |
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+-+
|            | SYSTEM       | CATALOG     | SYSTEM TABLE  |          |            |                            |                 | |
|            | SYSTEM       | FUNCTION    | SYSTEM TABLE  |          |            |                            |                 | |
|            | SYSTEM       | LOG         | SYSTEM TABLE  |          |            |                            |                 | |
|            | SYSTEM       | SEQUENCE    | SYSTEM TABLE  |          |            |                            |                 | |
|            | SYSTEM       | STATS       | SYSTEM TABLE  |          |            |                            |                 | |
+------------+--------------+-------------+---------------+----------+------------+----------------------------+-----------------+-+
0: jdbc:phoenix:localhost:2181> 
0: jdbc:phoenix:localhost:2181> create table test (mykey integer not null primary key, mycolumn varchar);
No rows affected (0.903 seconds)
0: jdbc:phoenix:localhost:2181> select * from test;
+--------+-----------+
| MYKEY  | MYCOLUMN  |
+--------+-----------+
+--------+-----------+
No rows selected (0.086 seconds)
0: jdbc:phoenix:localhost:2181> upsert into test values (1,'Hello');
1 row affected (0.393 seconds)
0: jdbc:phoenix:localhost:2181> upsert into test values (2,'World!');
1 row affected (0.007 seconds)
0: jdbc:phoenix:localhost:2181> select * from test;
+--------+-----------+
| MYKEY  | MYCOLUMN  |
+--------+-----------+
| 1      | Hello     |
| 2      | World!    |
+--------+-----------+
2 rows selected (0.02 seconds)
0: jdbc:phoenix:localhost:2181> 
```


#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

