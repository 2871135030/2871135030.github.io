---
layout: post
title: 使用jekyll的docker镜像运行项目
date:   2021-11-12 14:15:14
categories: [tools]
---

#### 使用jekyll的docker镜像运行项目

> jekyll镜像运行本项目所需的依赖版本不一致，用户权限，插件安装的问题需要特殊处理

* 下载镜像

```
docker pull jekyll/jekyll
```

* 运行镜像查看版本

```
docker run -it  --rm jekyll/jekyll bash
进入容器后 export 可查看到 jekyll 的版本如下
declare -x JEKYLL_VERSION="4.1.0"
```

* 上传网站目录

```
/home/jekyll/www
```

* 执行以下命令

```
docker run -it --rm -p 4000:4000 -v /home/jekyll/www/:/srv/jekyll/ jekyll/jekyll:4.1.0 jekyll server -w

```

> 出现错误`Could not find gem 'github-pages' in any of the gem sources listed in your Gemfile. `，原因是未安装`github-pages`

* 更换一个临时目录进入容器安装

```
docker run -it --rm -p 4000:4000 -v /home/jekyll/www/temp:/srv/jekyll/ jekyll/jekyll:4.1.0 jekyll server -w
```

> 又出现`Permission denied @ dir_s_mkdir - /srv/jekyll/.jekyll-cache`错误，原因是没有权限，根据Dockerfile可以看到需要使用jekyll用户进行操作

* 新建jekyll用户

```
adduser jekyll --password jekyll000

chmod -v u+w /etc/sudoers 修改为可编辑

vi /etc/sudoers 在root下添加一行jekyll
root    ALL=(ALL)       ALL
jekyll  ALL=(ALL)       ALL

chmod -v u-w /etc/sudoers 再修改为不可编辑

sudo groupadd docker 添加docker用户组
sudo gpasswd -a jekyll docker 将jekyll用户加入到docker用户组中
newgrp docker 更新用户组

chown -R jekyll:jekyll /home/jekyll/ 更新jekyll所有文件所属 

su jekyll 使用jekyll登录继续下一步

```

* 再次指定到临时目录运行

```
bash-4.2$ docker run -it --rm -p 4000:4000 -v /home/jekyll/www/temp:/srv/jekyll/ jekyll/jekyll:4.1.0 jekyll server -w
ruby 2.7.1p83 (2020-03-31 revision a0c7c23c9c) [x86_64-linux-musl]
Configuration file: none
            Source: /srv/jekyll
       Destination: /srv/jekyll/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    done in 0.012 seconds.
 Auto-regeneration: enabled for '/srv/jekyll'
    Server address: http://0.0.0.0:4000
  Server running... press ctrl-c to stop.

```

> 可以看到能够正常运行，接下来在此容器上安装`github-pages`

* 新打开一个窗口并进入刚刚启动的容器，由于本机只运行一个容器，可使用以下命令

```
docker exec -it $(docker ps -q) bash
```

* 进入容器后执行以下命令

```
bash-5.0# gem source
*** CURRENT SOURCES ***

https://rubygems.org/
bash-5.0# gem sources --remove https://rubygems.org/
https://rubygems.org/ removed from sources
bash-5.0# gem source -a https://gems.ruby-china.com/
https://gems.ruby-china.com/ added to sources
```

> 目的是更换gem国内源

* 安装github-pages插件

```
gem install github-pages

安装完成后可检查是否安装成功
bash-5.0# gem list --local |grep github-pages
github-pages (209)
github-pages-health-check (1.16.1)
```

* 退出进入的容器，并将已安装好github-pages插件的容器提交，成为新的镜像

```
docker commit $(docker ps -q) jekyll/jekyll:4.1.0-fix
```

* 停止jekyll用户运行`docker run`命令，将镜像改为`jekyll/jekyll:4.1.0-fix`再次启动

```
docker run -it --rm -p 4000:4000 -v /home/jekyll/www/:/srv/jekyll/ jekyll/jekyll:4.1.0-fix jekyll server -w
```

> 又出现错误`You have already activated i18n 1.8.5, but your Gemfile requires i18n 0.9.5. Prepending `bundle exec` to your command may solve this.`提示版本不匹配，将`Gemfile`文件放入目录`/home/jekyll/www/temp`，所以再次切换到temp目录运行容器

```
docker run -it --rm -p 4000:4000 -v /home/jekyll/www/temp:/srv/jekyll/ jekyll/jekyll:4.1.0-fix jekyll server -w
```

* 在新窗口使用以下命令再次进入容器处理版本不匹配问题

```
docker exec -it $(docker ps -q) bash

进入容器后执行
bash-5.0# ls
Gemfile  _site
bash-5.0# bundle update

```

* 退出进入的容器，并将已安装好github-pages插件的容器提交，成为新的镜像

```
docker commit $(docker ps -q) jekyll/jekyll:4.1.0-fix-v2
```

* 再次使用新的镜像进入就正常了

```
 docker run -it --rm -p 4000:4000 -v /home/jekyll/www:/srv/jekyll/ jekyll/jekyll:4.1.0-fix-v2 jekyll server -w
```





#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

