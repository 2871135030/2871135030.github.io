---
layout: post
title: centos使用rinetd实现端口转发
date:   2021-11-12 14:15:07
categories: [linux]
---

#### centos使用rinetd实现端口转发
##### 安装rientd

1、下载rientd安装包`wget https://github.com/samhocevar/rinetd/releases/download/v0.70/rinetd-0.70.tar.gz`

2、解压到安装目录/software，`tar -zxvf rinetd-0.70.tar.gz`

3、安装相关依赖`yum -y install gcc gcc-c++ make`

4、配置并安装，执行以下命令

```
./configure
make & make install
rinetd -v 可检查安装版本
```

##### 配置启动rientd
1、编辑配置文件/etc/rinetd.conf如下

```
192.168.2.5 2182 127.0.0.1 2181
```
表示将2182端口转发到2181
2、启动rientd

```
rinetd -c /etc/rinetd.conf
```


#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

