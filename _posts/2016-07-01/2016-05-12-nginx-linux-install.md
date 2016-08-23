---
layout: post
title: Linux环境下安装nginx
date:   2016-05-12 13:33:48
categories: [linux,tools]
---

## Linux环境下安装nginx


从nginx官网(http://nginx.org/en/download.html)下载nginx-1.11.3.tar.gz安装包

解压到目录

```markdown
cd /software
tar -xzvf nginx-1.11.3.tar.gz

```

安装nginx所需依赖，安装prce(重定向支持)和openssl(https支持，如果不需要https可以不安装。)

```markdown
yum -y install gcc
yum -y install pcre*
yum -y install openssl*
```


进入nginx目录安装

```markdown
cd /software/nginx-1.11.3

./configure --prefix=/user/local/nginx-1.11.3 --conf-path=/user/local/nginx-1.11.3/nginx.conf --with-http_ssl_module --with-http_stub_status_module --with-pcre

make & make install

注:nginx包解压目录为/software/nginx-1.11.3，安装目录为/user/local/nginx-1.11.3，nginx生成的配置文件为nginx.conf


cd /user/local/nginx-1.11.3
./sbin/nginx 即可启动nginx

重新加载配置及停止
./sbin/nginx -s reload
./sbin/nginx -s stop
	
```

结束。
