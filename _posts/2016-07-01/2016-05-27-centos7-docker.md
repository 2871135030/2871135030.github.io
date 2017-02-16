---
layout: post
title: Centos7安装使用docker部署tomcat应用笔记
date:   2016-05-27 13:59:58
categories: [tools]
---

## Centos7安装使用docker部署tomcat应用笔记

本文环境为vmware中安装centos7，然后在此centos中安装docker完成测试。

1、安装centos7(版本为CentOS-7-x86_64-Minimal-1611.iso)，先在vmware创建空白机器，再指定iso位置完成安装，安装步骤在些不再详述。

2、配置网络，修改文件/etc/sysconfig/network-scripts/ifcfg-enxxxx(xxxx各机器不一样，以实际为准)，将此文件的配置项ONBOOT修改为yes。
使用命令 service network restart 重启网络，再使用ip addr命令查看ip配置情况。

3、此版本的centos中Docker 软件包已经包括在默认的 CentOS-Extras 软件源里。因此想要安装 docker，只需要运行下面的 yum 命令  yum install docker -y 慢长等待下载完成安装即可，安装成功后可使用docker version查看版本。

```markdown
[root@localhost ~]# docker version
Client:
 Version:         1.12.5
 API version:     1.24
 Package version: docker-common-1.12.5-14.el7.centos.x86_64
 Go version:      go1.7.4
 Git commit:      047e51b/1.12.5
 Built:           
 OS/Arch:         linux/amd64

Server:
 Version:         1.12.5
 API version:     1.24
 Package version: docker-common-1.12.5-14.el7.centos.x86_64
 Go version:      go1.7.4
 Git commit:      047e51b/1.12.5
 Built:           
 OS/Arch:         linux/amd64
[root@localhost ~]# 
```

4、使用 systemctl start  docker.service 启动docker服务，使用systemctl stop  docker.service停止docker服务。
同时可使用命令systemctl enable docker.service将docker服务设置为开机启动，使用systemctl disable docker.service取消开机启动。ps -ef|grep docker可查看docker进程。

```markdown
[root@localhost ~]# ps -ef|grep docker
root     10580     1  0  ?        00:00:28 /usr/bin/dockerd-current --add-runtime docker-runc=/usr/libexec/docker/docker-runc-current --default-runtime=docker-runc --exec-opt native.cgroupdriver=systemd --userland-proxy-path=/usr/libexec/docker/docker-proxy-current --selinux-enabled --log-driver=journald --signature-verification=false
root     10585 10580  0  ?        00:00:03 /usr/bin/docker-containerd-current -l unix:///var/run/docker/libcontainerd/docker-containerd.sock --shim docker-containerd-shim --metrics-interval=0 --start-timeout 2m --state-dir /var/run/docker/libcontainerd/containerd --runtime docker-runc --runtime-args --systemd-cgroup=true
root     16587 15687  0  pts/0    00:00:00 grep --color=auto docker
[root@localhost ~]# 
```

5、现在docker已经安装好，启动docker服务，直接在线pull一个tomcat镜像完成测试，使用命令docker pull tomcat，又是漫长的等待后完成镜像下载。
下载完成后，可使用命令docker images查看本地已经存在的镜像。

```markdown
[root@localhost ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/tomcat    latest              d9094b6afb20        1 days ago          355.3 MB
[root@localhost ~]# 

```

6、基于已下载的tomcat镜像，创建一个容器，命令如下

```markdown
docker create --name container-tomcat -p 8080:8080 tomcat
// --name 给这个容器起一个名字
// -p host到container的端口映射
打一个比方说，一个image就相当于一个系统光盘，容器，就是一部安装了这个系统电脑。
```

使用命令docker start container-tomcat启动容器中的tomcat，然后使用docker ps可以查看到容器中的tomcat进程，使用docker stop container-tomcat关闭，或docker kill container-tomcat

```markdown
[root@localhost ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                    NAMES
74d19870e22c        tomcat              "catalina.sh run"   23 hours ago        Up 11 seconds       0.0.0.0:8080->8080/tcp   container-tomcat
[root@localhost ~]# 
```


容器启动后可以用docker logs -f container-tomcat 查看跟踪日志
若需要重启容器使用docker restart container-tomcat即可。

7、启动容器后即可访问tomcat(container的端口8080已映射到host的8080)。 可以直接url http://localhost:8080进行访问，查看结果。外部直接使用宿主host的ip进行访问即可。

8、前面已经看到docker容器里的tomcat已经启动了，那么，我们要怎么查看管理到容器里的tomcat呢？

```markdown
使用以下命令进入容器
sudo docker exec -it 74d19870e22c /bin/bash 
或
sudo docker exec -it container-tomcat /bin/bash
```

如下，进入到容器后pwd路径为容器内的目录路径，使用exit退出。

```markdown
[root@localhost ~]# docker exec -it container-tomcat /bin/bash
root@74d19870e22c:/usr/local/tomcat# pwd
/usr/local/tomcat
root@74d19870e22c:/usr/local/tomcat# 
```

比如要使用vi修改/usr/local/tomcat/conf/tomcat-users.xml配置文件，使tomcat的manager生效，先使用命令
apt-get update，这个命令的作用是：同步 /etc/apt/sources.list 和 /etc/apt/sources.list.d 中列出的源的索引，这样才能获取到最新的软件包。
再安装vim，使用命令apt-get install vim 然后就可以使用vi命令操作了。将tomcat-users.xml原用户配置部分取消注释并设置好密码，同时必须添加一个用户角色<role rolename="manager-gui"/>，并将此角色添加到对应的用户上即可。

9、如何将自己的war包上传到容器中的tomcat webapp下呢？
从container到主机的复制如下
docker cp container-tomcat:/usr/local/tomcat/conf/tomcat-users.xml /war/tomcat-users.xml
从主机到container的复制如下
docker cp /war/simpleweb.war container-tomcat:/usr/local/tomcat
上传war包后就可以完成部署到docker中tomcat了。

docker restart container-tomcat 重启tomcat即可生效。

结束。
