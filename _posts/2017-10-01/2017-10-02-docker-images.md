---
layout: post
title: docker创建与使用镜像
date:   2017-10-02 14:31:58
categories: [spring,tools]
---

## docker创建与使用镜像

1、接上一篇<a href="/spring/tools/2017/10/01/docker-install.html">docker安装与基本使用</a>，我们已经装好docker环境了，接下来看看怎么使用它了，可以直接到镜像仓库拉取直接使用
{% highlight ruby %}
可先查找镜像，如查找一个jdk8的镜像
docker search jdk8
找到合适的镜像后，可直接拉取，其中xxx为镜像名
docker pull xxx
{% endhighlight %}

2、直接在线拉取镜像使用本文不作介绍，下面动手创建一个jdk8的基本环境的镜像，当然这个jdk8环境是基于centos镜像的，
{% highlight ruby %}
mkdir /docker
cd /docker
{% endhighlight %}
在docker目录中编写一个文件Dockerfile，内容如下
{% highlight ruby %}
FROM centos
MAINTAINER mixfate
ADD jdk-8u65-linux-x64.tar.gz /
ENV JAVA_HOME=/jdk1.8.0_65
ENV PATH=$JAVA_HOME/bin:$PATH
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
EXPOSE 8080
ENTRYPOINT ["java","-version",""]
{% endhighlight %}
将jdk上传到/docker/jdk-8u65-linux-x64.tar.gz

3、执行命令build构建镜像，结果如下
{% highlight ruby %}
[root@localhost docker]# docker build -t java1.8 .
Sending build context to Docker daemon  181.3MB
Step 1/8 : FROM centos
 ---> af7c74ac94e5
Step 2/8 : MAINTAINER mixfate
 ---> Using cache
 ---> 1005831b27e6
Step 3/8 : ADD jdk-8u65-linux-x64.tar.gz /
 ---> Using cache
 ---> 662ec0354395
Step 4/8 : ENV JAVA_HOME /jdk1.8.0_65
 ---> Running in 62297aa627b4
 ---> b1ffd98d6652
Removing intermediate container 62297aa627b4
Step 5/8 : ENV PATH $JAVA_HOME/bin:$PATH
 ---> Running in 5f6f1a254789
 ---> 50a8db84c4f5
Removing intermediate container 5f6f1a254789
Step 6/8 : ENV CLASSPATH .:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
 ---> Running in 3f0b61c3982b
 ---> 70d9c1b97d25
Removing intermediate container 3f0b61c3982b
Step 7/8 : EXPOSE 8080
 ---> Running in 8abbd2583387
 ---> d096ed08f5f5
Removing intermediate container 8abbd2583387
Step 8/8 : ENTRYPOINT java -version 
 ---> Running in 558d376716be
 ---> b7ff86b0c1bb
Removing intermediate container 558d376716be
Successfully built b7ff86b0c1bb
Successfully tagged java1.8:latest
[root@localhost docker]#
{% endhighlight %}

4、此时可使用命令docker images查看构建好的镜像 java1.8
{% highlight ruby %}
[root@localhost docker]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
java1.8             latest              fcd857ec1a79        48 seconds ago      561MB
centos              latest              af7c74ac94e5        10 minutes ago      197MB
[root@localhost docker]# 
{% endhighlight %}

5、有了镜像就可以创建容器了，命令如下
{% highlight ruby %}
[root@localhost docker]# docker run -i -p 8080:8080 -t java1.8 /bin/bash
java version "1.8.0_65"
Java(TM) SE Runtime Environment (build 1.8.0_65-b17)
Java HotSpot(TM) 64-Bit Server VM (build 25.65-b01, mixed mode)
[root@localhost docker]#
{% endhighlight %}

6、常用操作命令
{% highlight ruby %}
进入容器
docker exec -it java1.8 /bin/bash
使用 Ctrl + P + Q 退出容器

停止运行容器
docker stop xxx
docker stop $(docker ps -q) 停止所有容器

从容器中导出镜像文件及导入
docker export xxxx > java1.8.tar
cat java1.8.tar | docker import - java1.8
其中 xxxx 为容器id

从镜像中导出镜像文件及导入
docker save xxxx > centos.tar
docker load < centos.tar
其中 xxxx 为镜像id

注意
需注意两种方法不可混用，虽然导入不提示错误，但是启动容器时会提示失败。

删除容器及删除镜像
docker rm xxx 删除容器
docker rm $(docker ps -a -q) 删除所有容器
docker rmi xxx 删除镜像


{% endhighlight %}


结束。
