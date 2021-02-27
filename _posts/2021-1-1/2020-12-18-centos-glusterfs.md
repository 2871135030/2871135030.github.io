---
layout: post
title: centos7安装glusterfs
date:   2021-11-12 14:15:23
categories: [tools]
---

#### centos7安装glusterfs

##### 1、环境机器准备

> 准备四台机器，`host`配置如下，`gfs01/gfs02/gfs03`为gluster节点均需执行`systemctl stop firewalld`关闭防火墙或`firewall-cmd --zone=public --add-service=glusterfs --permanent`并`firewall-cmd --reload`

```
192.168.80.136 gfs01
192.168.80.142 gfs02
192.168.80.143 gfs03
192.168.80.144 gfs-client
```

##### 2、gfs01\gfs02\gfs03三台节点机器新增一块硬盘并格式化(在vmware中实践)

* 检查系统中是否有新磁盘

```
fdisk -l
```

> 可以查看到是否存在`Disk /dev/sda`、`Disk /dev/sdb`

* 执行分区命令对新硬盘`/dev/sdb`进行分区

```
fdisk /dev/sdb

Command (m for help): n
输入n新建分区

Select (default p): p
输入p新增为主分区

Command (m for help): p
输入p可查看详情

Command (m for help): w
输入w保存变更
```

* 执行分区格式化

```
mkfs.ext4 /dev/sdb1
```

* 执行挂载命令并查看挂载情况

```
mkdir /sdb1
mount /dev/sdb1 /sdb1
df -Th
```

> 重启后会丢失，需将挂载添加到配置中`echo  "/dev/sdb1 /sdb1 ext4 defaults 0 0" >> /etc/fstab`

##### 3、gfs01/gfs02/gfs03三台机器均通过yum安装glusterfs

* 安装glusterfs源

```
yum install -y epel-release

搜索glusterfs版本
yum search centos-release-gluster

这里选择gluster6的版本进行安装
yum install centos-release-gluster6 -y

```

* 安装glusterfs软件包

```
yum -y install glusterfs glusterfs-server glusterfs-fuse glusterfs-rdma
```

##### 4、启动glusterfsd

```
systemctl start glusterfsd
systemctl status glusterfsd
```

##### 5、在任一节点上操作添加另外两个节点（在gfs01上操作）

* 在gfs01节点上操作添加gfs01/gfs03节点（添加时需保证 glusterfsd系统服务已经启动）

```
[root@gfs01 /]# gluster peer probe gfs01 #本机无需操作
peer probe: success. Probe on localhost not needed
[root@gfs01 /]# gluster peer probe gfs02
peer probe: success. 
[root@gfs01 /]# gluster peer probe gfs03
peer probe: success. 
[root@gfs01 /]# netstat -antp|grep gluster
tcp        0      0 0.0.0.0:24007           0.0.0.0:*               LISTEN      8484/glusterd       
tcp        0      0 192.168.80.136:24007    192.168.80.142:49151    ESTABLISHED 8484/glusterd       
tcp        0      0 192.168.80.136:49150    192.168.80.143:24007    ESTABLISHED 8484/glusterd       
tcp        0      0 192.168.80.136:24007    192.168.80.143:49151    ESTABLISHED 8484/glusterd       
tcp        0      0 192.168.80.136:49151    192.168.80.142:24007    ESTABLISHED 8484/glusterd       
[root@gfs01 /]# 
```

* 检查gluster集群状态

```
[root@gfs01 /]# gluster peer status 
Number of Peers: 2

Hostname: gfs02
Uuid: 0cc7b46d-fa95-4cbe-8531-da28d10b58cf
State: Peer in Cluster (Connected)

Hostname: gfs03
Uuid: fe0882bc-cde4-43de-bd49-d1c542eb65bb
State: Peer in Cluster (Connected)
[root@gfs01 /]# gluster pool list
```

> 在其中一个节点中可以看到另外两个节点信息（如在gfs01上可以看到gfs02及gfs03的信息，gfs02/gfs03同理）

##### 6、创建分布式卷(Distribute)

* 创建distribute-volume卷作示例（默认创建的类型）

```
[root@gfs03 ~]# gluster volume create distribute-volume gfs01:/sdb1 gfs02:/sdb1 gfs03:/sdb1 force
volume create: distribute-volume: success: please start the volume to access data
[root@gfs03 ~]# gluster volume list
distribute-volume
[root@gfs03 ~]# gluster volume info distribute-volume
 
Volume Name: distribute-volume
Type: Distribute # 可以看到这个类型为分布式卷
Volume ID: b0087b4c-d9a3-47be-aafa-982f2ad5a21f
Status: Created
Snapshot Count: 0
Number of Bricks: 3
Transport-type: tcp
Bricks: # 共有三个brick存储服务器
Brick1: gfs01:/sdb1
Brick2: gfs02:/sdb1
Brick3: gfs03:/sdb1
Options Reconfigured:
transport.address-family: inet
nfs.disable: on
[root@gfs03 ~]#
```

* 创建之后的卷需要要启用后才能使用，启动分布式卷

```
[root@gfs03 ~]# gluster volume start distribute-volume
[root@gfs03 ~]# gluster volume status
```

> 分布式卷启动之后可以通过`gluster volume info distribute-volume`查看到`Status`状态为`Started`，说明已经启动成功；

##### 7、安装客户端

> 在部署完分布式文件系统后，在需要使用glusterfs服务器上安装相应的客户端软件，并创建挂载目录， 将分布式文件系统挂载到刚创建目录即可。

* 安装工具包

```
yum -y install glusterfs glusterfs-fuse
```

* 执行目录挂载

```
[root@gfs-client /]# mkdir -p /data/distribute-volume
[root@gfs-client /]# mount.glusterfs gfs01:distribute-volume /data/distribute-volume
[root@gfs-client /]# df -Th
Filesystem              Type            Size  Used Avail Use% Mounted on
......
gfs01:distribute-volume fuse.glusterfs   59G  734M   56G   2% /data/distribute-volume
```

> 如果需要重启后自动挂载`echo "gfs01:distribute-volume /data/distribute-volume glusterfs defaults 0 0" >> /etc/fstab`

* 在`/data`目录中产生一些测试文件

```
[root@gfs-client /]# cd /data/
[root@gfs-client data]# dd if=/dev/zero of=test1.log bs=1M count=10 
```

> 多次执行dd命令产生测试文件test1.log、test2.log、test3.log、test4.log，然后将所有test文件移动到distribute-volume中，检查存储情况，因为创建的是分布式卷，所以文件会被在其中一个节点保存；

> 在演示中gfs01的目录/sdb1中保存了test3.log，gfs02的目录/sdb1中保存了test1.log和test1.log，gfs03的目录/sdb1中保存了test4.log

##### 8、实践glusterfs的复制卷类型

* 创建复制卷(Distribute Volume)

```
fdisk /dev/sdc
mkfs.ext4 /dev/sdc1
echo "/dev/sdc1 /sdc1 ext4 defaults 0 0" >> /etc/fstab 
mount -a
```

```
gluster volume create replica-volume replica 3 gfs01:/sdc1 gfs02:/sdc1 gfs03:/sdc1 force
3表示文件分成3个副本存储

gluster volume start replica-volume

gluster volume status
```

* 在客户端的机器挂载并测试

```
echo "gfs01:replica-volume /data/replica-volume glusterfs defaults 0 0" >> /etc/fstab
mount -a

dd if=/dev/zero of=test-replica.log bs=1M count=10 
```




#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

