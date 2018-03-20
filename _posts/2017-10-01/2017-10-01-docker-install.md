---
layout: post
title: docker安装与基本使用
date:   2017-10-01 13:11:58
categories: [spring,tools]
---

## docker安装与基本使用

1、由于在低版本的环境中安装docker出现诸多问题，一一解决费时费力，本文centos以镜像CentOS-7-x86_64-Minimal-1611.iso安装，在centos7中安装docker-ce公开版
首先使用以下命令删除旧版本
{% highlight ruby %}
sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
{% endhighlight %}
然后更新系统
{% highlight ruby %}
yum update
{% endhighlight %}

2、为了清楚了解依赖，先下载离线安装包，先安装离线下载工具，安装完后可使用命令yumdownloader查看
{% highlight ruby %}
yum install yum-utils
{% endhighlight %}

3、配置好下载地址（不保证此地址一直可用），离线下载
{% highlight ruby %}
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
{% endhighlight %}
分别执行以下使用，完成下载docker-ce公开版及相关依赖
{% highlight ruby %}
mkdir /software
cd /software
sudo yumdownloader --resolve docker-ce
sudo yumdownloader --resolve container-selinux
sudo yumdownloader --resolve 'libltdl.so.7()(64bit)'
{% endhighlight %}

4、下载完成后执行离线安装，命令如下
{% highlight ruby %}
rpm -i audit-libs-python-2.7.6-3.el7.x86_64.rpm
rpm -i checkpolicy-2.5-4.el7.x86_64.rpm
rpm -i libcgroup-0.41-13.el7.x86_64.rpm
rpm -i libsemanage-python-2.5-8.el7.x86_64.rpm
rpm -i libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
rpm -i python-IPy-0.75-6.el7.noarch.rpm
rpm -i setools-libs-3.3.8-1.1.el7.x86_64.rpm
rpm -i policycoreutils-python-2.5-17.1.el7.x86_64.rpm
rpm -i container-selinux-2.28-1.git85ce147.el7.noarch.rpm
rpm -i docker-ce-17.09.0.ce-1.el7.centos.x86_64.rpm
{% endhighlight %}

5、成功安装后可使用以下命令检查
{% highlight ruby %}
[root@localhost software]# yum list installed|grep docker
docker-ce.x86_64                      17.09.0.ce-1.el7.centos          installed
[root@localhost software]# 
{% endhighlight %}

6、启动docker命令
{% highlight ruby %}
systemctl start docker
{% endhighlight %}

7、docker常用命令
{% highlight ruby %}
查看docker镜像 
docker images
删除镜像
docker rmi 镜像名
{% endhighlight %}
结束。
