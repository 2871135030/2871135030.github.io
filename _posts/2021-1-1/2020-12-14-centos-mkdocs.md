---
layout: post
title: centos7安装mkdocs
date:   2021-11-12 14:15:19
categories: [tools]
---

#### centos7安装mkdocs

##### 1、检查版本并下载安装python3.8.2

* 版本信息

```
[root@localhost ~]# cat /etc/redhat-release 
CentOS Linux release 7.9.2009 (Core)
[root@localhost ~]# uname -a
Linux localhost.localdomain 3.10.0-1160.11.1.el7.x86_64 #1 SMP Fri Dec 18 16:34:56 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
[root@localhost ~]# 
```

* 安装依赖

```
yum install -y gcc openssl-devel bzip2-devel libffi-devel zlib-devel
```

* 下载并解压`python3`完成安装

```
解压python3.8.2安装包
tar -zxvf Python-3.8.2.tgz
cd Python-3.8.2
./configure --prefix=/usr/local/python3
make && make install 
```

* 安装完成后创建软链并检查安装好的版本

```
[root@localhost Python-3.8.2]# ln -s /usr/local/python3/bin/python3 /usr/bin/python3
[root@localhost Python-3.8.2]# ln -sf /usr/local/python3/bin/python3 /usr/bin/python
[root@localhost Python-3.8.2]# ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3
[root@localhost Python-3.8.2]# ln -s /usr/local/python3/bin/pip3 /usr/bin/pip
[root@localhost Python-3.8.2]# python3 --version 
Python 3.8.2
[root@localhost Python-3.8.2]# pip3 -V
pip 19.2.3 from /usr/local/python3/lib/python3.8/site-packages/pip (python 3.8)
```

##### 2、安装`mkdocs`

* 配置`pip`国内源加快下载资源包的速度

```
[root@localhost Python-3.8.2]# more ~/.pip/pip.conf 
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host = https://pypi.tuna.tsinghua.edu.cn
[root@localhost Python-3.8.2]# 
```

* 执行安装相关`mkdocs`及相关依赖

```
pip install --upgrade pip
pip install wheel
pip install mkdocs mdx_gh_links mkdocs-material   --target=/usr/local/mkdocs

ln -s /usr/local/mkdocs/bin/mkdocs  /usr/bin/mkdocs

```

* 检查安装好的`mkdocs`版本

```
[root@localhost ~]# mkdocs --version
mkdocs, version 1.1.2 from /usr/local/python3/lib/python3.8/site-packages/mkdocs (Python 3.8)
[root@localhost ~]# 
```

##### 3、使用`mkdocs`

* 创建新的项目

```
[root@localhost project]# mkdocs new free
INFO    -  Creating project directory: free 
INFO    -  Writing config file: free/mkdocs.yml 
INFO    -  Writing initial docs: free/docs/index.md 
```

> 默认生成了配置文件`mkdocs.yml`及`docs`目录下的`index.md`页面，自己创建的页面需放到`docs`目录下，在`nav`中支持多级目录书写

* 启动应用

```
[root@localhost ~]# cd /project/free/
[root@localhost free]# mkdocs serve -a 192.168.80.130:8000
INFO    -  Building documentation... 
INFO    -  Cleaning site directory 
INFO    -  Documentation built in 0.06 seconds 
```

> 启动后即可通过`http://192.168.80.130:8000`访问页面

* 自定义配置文件

```
site_name: FreeDocs

nav:
  - Home: index.md
  - Work: 
    - November: november.md
    - December: december.md
  - about: about.md
theme: material
# theme: readthedocs
# theme: gitbook
```

> 可以安装对应不同的主题，如`gitbook`，可使用`pip install mkdocs-gitbook`安装此主题

> 注意旧版的`pages`已弃用，已改为`nav`






#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

