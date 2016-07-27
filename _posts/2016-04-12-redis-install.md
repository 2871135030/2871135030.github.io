---
layout: post
title: redis安装使用
date:   2016-04-12 15:32:57
categories: [tools,redis]
---

## Redis安装使用

下载redis-2.8.19.tar.gz

tar -xzvf redis-2.8.19.tar.gz

进入目录 /software/redis-2.8.19

make 完成安装

若出现错误gcc：命令未找到，先执行 yum -y install gcc

若出现错误：jemalloc/jemalloc.h：没有那个文件或目录，执行 make MALLOC=libc 完成安装

安装成功后完成配置
{% highlight ruby %}
vi /software/redis-2.8.19/redis.conf 为redis设置密码 项 requirepass redis2016
mkdir /usr/redis
cp redis-server /usr/redis
cp redis-benchmark /usr/redis
cp redis-cli /usr/redis
cp ../redis.conf /usr/redis
cd /usr/redis
ps -ef | grep 'redis-server' | grep -v grep| awk '{print $2}'| xargs kill -9
./redis-server & 后台运行
{% endhighlight %}

结束。
