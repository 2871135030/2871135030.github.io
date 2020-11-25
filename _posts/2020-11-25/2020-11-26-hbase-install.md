---
layout: post
title: HBase2.3单机版安装使用
date:   2021-11-12 14:15:01
categories: [hbase]
---
#### HBase2.3单机版安装使用
##### 1、安装单机版HBase
```
下载hbase-2.3.0-bin.tar.gz及jdk-8u151-linux-x64.tar.gz并解压到目录/software
关闭防火墙、禁用ipv6并设置hostname为hbase，，同时需调整时区为中国时区

检查系统版本为centos7
[root@hbase ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)
[root@hbase ~]# uname -a
Linux hbase 3.10.0-1127.19.1.el7.x86_64 #1 x86_64 x86_64 x86_64 GNU/Linux
[root@hbase ~]# 
```

* 设置`jdk`及`hbase`环境变量，在`/etc/profile`文件末尾添加以下配置后使用`source /etc/profile`使其生效

```
export JAVA_HOME=/software/jdk1.8.0_151
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

export HBASE_HOME=/software/hbase-2.3.0
export PATH=$HBASE_HOME/bin:$PATH

```
* 检查安装情况，可查看jdk与hbase版本

```
[root@hbase bin]# pwd
/software/hbase-2.3.0/bin
[root@hbase bin]# java -version
java version "1.8.0_151"
Java(TM) SE Runtime Environment (build 1.8.0_151-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.151-b12, mixed mode)
[root@hbase bin]# ./hbase version
HBase 2.3.0
Source code repository git://35f72a441626/home/vagrant/hbase-rm/output/hbase revision=e0e1382705c59d3fb3ad8f5bff720a9dc7120fb8
Compiled by vagrant
From source with checksum (stdin)=
[root@hbase bin]# 
```

* 配置`hbase`

```
可查看配置文件/software/hbase-2.3.0/conf/hbase-env.sh默认配置项HBASE_MANAGES_ZK为true（表示使用hbase内置zookeeper，启动后可使用netstat -ano|grep 2181查看端口占用情况）

编辑配置文件/software/hbase-2.3.0/conf/hbase-site.xml，添加或修改以下配置 
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
</configuration>
  
```
* 启动`hbase`

```
[root@hbase bin]# pwd
/software/hbase-2.3.0/bin
[root@hbase bin]# ./start-hbase.sh 
running master, logging to /software/hbase-2.3.0/logs/hbase-root-master-hbase.out
[root@hbase bin]# jps
2000 Jps
1561 HMaster

可以查看到hbase已正常启动，netstat -ano|grep 2181可以看到内置zookeeper已启动，可以使用http://192.168.2.5:16010/master-status访问看到hbase控制台

```

 
##### 2、hbase常用命令操作

命令|说明|命令语法
-|-|-
status|查看hbase状态信息|-
tools|查看所有工具命令|-
version|查看hbase版本信息|-
list_namespace|查看命名空间（支持正则）|`查看所有list_namespace`<br>`正则匹配list_namespace 'ioe.*'`
create_namespace|创建命名空间|`create_namespace <namespace>`<br>`create_namespace 'ioe'`
alter_namespace|修改命名空间|`如修改namespace的创建人 alter_namespace 'ioe', {METHOD=>'set', 'create'=>'xiaopang'}`
list_namespace_tables|查看命名空间下的表|`list_namespace_tables<namespace>`<br>`list_namespace_tables 'ioe'`
describe_namespace|查看命名空间详细信息|-
drop_namespace|删除命名空间|`需先清除表drop_namespace <namespace>`
create|创建表（指定namespace）|`create <namespace>:<tableName>,<columnFamily1>,<columnFamily2> `<br>`create 'ioe:user','a','b' `
create|创建表（默认namespace）|`create <tableName>,<columnFamily1>,<columnFamily2>`<br> `create 'user','name','address'用户有中文名、英文名、小名，地址有家庭地址、办公地址、户籍地址`
list|查看所有表|`list`<br>`列出其他namespace表list 'ioe.*'`
describe|查看表详细信息|`describe <tableName>`<br> `describe 'user'`<br>`查看其他namespace表describe 'ioe:user'`
exists|判断表是否存在|`exists <tableName>`
enable|启用表|`enable <tableName>`
disable|停用表|`disable <tableName>`
disable_all|批量停用表|`disable '.*`
is_enabled|判断是否启用|`is_enabled <tableName>`
is_disabled|判断是否停用|`is_disabled <tableName>`
count|统计表中行数|`count <tableName>`
put|新增记录|`put <tableName>,<rowkey>,<columnFamily>:<column>,<value>` <br>`put 'user','xiaopang','name:en','michael'` <br>`put 'user','xiaopang','name:ch','胖子'`<br>`put 'user','xiaopang','address:office','北京'`
get|查询记录|`get <tableName>,<rowkey>` <br>`get 'user','xiaopang'`<br>`获取多个版本数据get 'user','xiaopang',{COLUMN=>'phone:a',VERSIONS=>3}`
delete|删除记录|`delete <tableName>,<rowkey>,<columnfamily>:<column>` <br>`delete 'user','xiaopang','name:ch'`
deleteall|删除整行记录|`deleteall <tableName> <rowkey>` <br>`deleteall 'user','xiaopang'`
drop|删除表（必须先disable）|`drop <tableName>`<br>`drop 'user'`
drop_all|批量删除表（必须先disable）|`drop_all '.*'`
alter|增加、修改、删除列簇|`增加alter <table>,<columnfamily>`<br>`修改alter <table>,<columnfamily>`<br>`删除alter <table>,{NAME=><columnfamily>,METHOD=>'delete'}`<br>`alter 'user','phone'`<br>`alter 'user',{NAME=>'phone',METHOD=>'delete'}`<br>`或alter 'user', 'delete' => 'phone'`<br>`修改为最多获取5个版本alter 'user', NAME=>'phone',VERSIONS=>5`
incr|字段自增|`incr 'user','xiaopang','age',9`
truncate|清除表数据|-
scan|扫描所有记录|`scan <tableName>` <br>`scan 'user'`
exit|退出| -
shutdown|关闭集群|-
 

#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

