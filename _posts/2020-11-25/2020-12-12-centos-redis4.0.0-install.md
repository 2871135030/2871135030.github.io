---
layout: post
title: centos7安装redis4.0.0
date:   2021-11-12 14:15:17
categories: [redis]
---

##### centos7安装redis4.0.0

###### 1、下载安装包并解压

```
wget https://github.com/redis/redis/archive/4.0.0.tar.gz
tar -zxvf 4.0.0.tar.gz
cd redis-4.0.0/
```

###### 2、安装依赖

```
yum install gcc tcl  -y
```

###### 3、安装并测试

```
make MALLOC=libc
make test
```

###### 4、修改配置

```
注释 bind绑定的ip
daemonize 改为 yes 后台运行
requirepass 修改密码
```

###### 5、配置为系统服务

* 编辑此文件`/lib/systemd/system/redis.service`，内容如下

```
[Unit]
Description=redis
After=network.target

[Service]
Type=forking
PIDFile=/var/run/redis_6379.pid
ExecStart=/software/redis-4.0.0/src/redis-server /software/redis-4.0.0/redis.conf
ExecReload=/bin/kill -s HUP MAINPIDExecStop=/bin/kill?sQUITMAINPID
PrivateTmp=true
```

* 重新加载配置，同时禁用防火墙

```
systemctl daemon-reload
systemctl stop firewalld
```



#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

