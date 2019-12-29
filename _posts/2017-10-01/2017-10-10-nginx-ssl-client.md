---
layout: post
title: Centos7环境使用Nginx自建双向认证
date:   2021-11-12 14:12:00
categories: [nginx]
---

## Centos7环境使用Nginx自建双向认证


1、配置双向认证基于已经搭建好单向认证，如果我们需要对客户端进行验证，让受信任的客户端即持有证书的客户端才能访问，这时我们就需要双向认证来处理，
使用自建CA生成证书


2、服务端证书生成
```
openssl genrsa -out ca.key 2048
openssl req -new -x509 -days 3650 -key ca.key -out ca.crt
openssl x509 -req -sha256 -in mixfate.csr -CA ca.crt  -CAkey ca.key  -CAcreateserial -days 3650 -out mixfate.crt
用CA签发原单向认证的服务端证书(客户端安装ca.crt到“受信任的根证书颁发机构”)
openssl x509 -req -sha256 -in mixfate.csr -extfile v3.ext -CA ca.crt  -CAkey ca.key  -CAcreateserial -days 3650 -out mixfate-server-client.crt
```

3、客户端证书
```
openssl genrsa -out client.pem 1024
openssl rsa -in client.pem -out client.key
openssl req -new -key client.pem  -out client.csr
用CA签发
openssl x509 -req -sha256 -in client.csr -CA ca.crt  -CAkey ca.key  -CAcreateserial -days 3650 -out client.crt
为了在windows的浏览器上演示，生成pfx格式的安装证书，client.pfx需在windows上安装
openssl pkcs12 -export -inkey client.key -in client.crt -out client.pfx
```


4、nginx配置文件中修改如下

```
    server {
        listen       443 ssl;
        server_name  www.mixfate.com;
        ssl_certificate mixfate-server-client.crt;
        ssl_certificate_key mixfate.key;
        ssl_client_certificate ca.crt;
        ssl_verify_client on;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
		...其余配置省略
	}
```
结束。。。
