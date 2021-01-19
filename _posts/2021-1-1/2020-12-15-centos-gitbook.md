---
layout: post
title: centos7安装gitbook
date:   2021-11-12 14:15:20
categories: [tools]
---

#### centos7安装gitbook

##### 1、安装node

* 下载node安装包并解压

```
wget  https://nodejs.org/dist/v10.16.3/node-v10.16.3-linux-x64.tar.xz

xz -d node-v10.16.3-linux-x64.tar.xz
tar -xvf node-v10.16.3-linux-x64.tar
```

* 创建软链

```
ln -s /software/node-v10.16.3-linux-x64/bin/node /usr/local/bin/node
ln -s /software/node-v10.16.3-linux-x64/bin/npm /usr/local/bin/npm

```

* 检查安装后的版本

```
[root@localhost software]# node -v
v10.16.3
[root@localhost software]# npm -v
6.9.0
[root@localhost software]# 
```

* 配置npm国内的镜像并检查是否生效

```
[root@localhost software]# npm config set registry https://registry.npm.taobao.org -g
[root@localhost software]# npm config get registry
https://registry.npm.taobao.org/
[root@localhost software]# 
```

##### 2、安装gitbook

* 使用npm安装gitbook

```
npm install gitbook-cli -g
```

* 创建软链

```
ln -s /software/node-v10.16.3-linux-x64/bin/gitbook /usr/local/bin/gitbook

```

* 检查gitbook版本（期间将会执行install）

```
[root@localhost software]# gitbook -V
CLI version: 2.3.2
Installing GitBook 3.2.3
```

> 可能耗时比较久，可打开另外一个命令窗口，使用命令`ps -ef|grep node|grep gitbook|awk '{print $2}'|xargs strace -ttp`观察安装情况，如果实在太慢可中断后再次开始，安装完成后如下

```
[root@localhost software]# gitbook -V
CLI version: 2.3.2
GitBook version: 3.2.3
[root@localhost software]# 
```

##### 3、使用gitbook

* 初始化gitbook

```
mkdir -p /project/gitbook
cd /project/gitbook
gitbook init
```

> 初始化完成后，可以看到目录下有文件`SUMMARY.md`，这个文件即用来配置目录；

```
[root@localhost gitbook]# more SUMMARY.md 
# Summary

* [Introduction](README.md)
* [11](11.md)
  * [1101](1101.md)
  * [1102](1102.md)
```

* 配置基本参数（新建book.json文件）

```
[root@localhost gitbook]# more book.json 
{
    "title": "GitbookDemo",
    "author": "free",
    "description": "Gitbook Demo",
    "links": {
        "sidebar": {
            "Home": "https://www.free.com"
        }
    },
    "styles": {
        "website": "styles/website.css",
        "ebook": "styles/ebook.css",
        "pdf": "styles/pdf.css",
        "mobi": "styles/mobi.css",
        "epub": "styles/epub.css"
    },
    "plugins": [
        "-search",
        "highlight",
        "back-to-top-button",
        "expandable-chapters-small",
        "insert-logo",
        "code",
        "donate"
    ],
    "pluginsConfig": {
        "insert-logo": {
            "url": "images/logo.png",
            "style": "background: none; max-height: 30px; min-height: 30px"
        },
        "donate": {
            "wechat": "donate-wechatpay.png",
            "alipay": "donate-alipay.png",
            "title": "",
            "button": "赏",
            "alipayText": "支付宝打赏",
            "wechatText": "微信打赏"
        }
    }
}
```

* 安装对应的插件

```
gitbook install
```

* 启动服务

```
gitbook serve
```

> 服务启动后即可通过`http://192.168.80.130:4000`访问






#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

