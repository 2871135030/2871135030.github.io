---
layout: post
title: 阿里云ECS服务器密钥对pem文件使用secureCRT登录异常问题处理
date:   2020-11-15 12:11:09
categories: [spring]
---

#####  阿里云ECS服务器密钥对pem文件使用secureCRT登录异常问题处理

密钥对pem文件非公钥文件，并不能直接在`SecureCRT`客户端中直接使用，会报`Unable to authenticate using any of the configured authentication methods.`错误。

###### 1、将下载的pem文件放到`linux`环境下，使用以下命令导出公钥

```
chmod 600 mixfate.pem
ssh-keygen -e -f mixfate.pem > mixfate.pem.pub
```

###### 2、再将mixfate.pem.pub公钥导入secureCRT

> Session Options -> Connection(SSH2) -> Authentication(PublicKey) -> Properties -> Use identity or certificate file，选中公钥文件即可。
