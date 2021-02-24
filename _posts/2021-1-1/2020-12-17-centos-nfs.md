---
layout: post
title: centos7安装nfs服务
date:   2021-11-12 14:15:22
categories: [tools]
---

#### centos7安装nfs服务

##### 1、服务端安装

* 安装软件包

```
yum -y install nfs-utils rpcbind
```

* 修改配置文件`/etc/exports`

```
/data *(rw,async,no_root_squash)
```

> 在根目录新建目录`data`，并赋权限`chomd -R a+w /data`

* 防火墙设置

```
开放111及2049端口
firewall-cmd --zone=public --add-port=111/tcp --permanent
firewall-cmd --zone=public --add-port=2049/tcp --permanent
firewall-cmd --reload
检查开放的端口是否生效
firewall-cmd --list-port
```

* 启动服务

```
systemctl start rpcbind
systemctl start nfs
```

> 检查`rpcbind`端口是否正常`netstat -lntup|grep rpcbind`

> 检查`nfs`端口是否正常`netstat -lntup|grep 2049`

* 本机挂载测试

```
mkdir /nfs
mount -t nfs 192.168.80.138:/data /nfs
其中192.168.80.138为服务端ip
```

> 将`nfs`挂载到本地目录`/nfs`，再通过`df -h`即可查看到挂载情况


##### 2、客户端安装

* 安装软件包

```
yum  -y  install  nfs-utils
```

* 挂载`nfs`

```
mkdir /nfs
mount 192.168.80.138:/data /nfs
df -h 
重启后会丢失需重新挂载，增加开机自动挂载
echo  "192.168.80.138:/data /nfs nfs defaults 0 0" >> /etc/fstab
```







#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

