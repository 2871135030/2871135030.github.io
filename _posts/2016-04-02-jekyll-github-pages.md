---
layout: post
title: Windows环境jekyll使用
date:   2016-04-02 23:51:39
categories: tools
---

## Windows环境jekyll使用

需准备的软件如下：
![jekyll](/assets/20160617230019.png)

1、安装rubyinstaller-2.0.0-p648-x64，假如目录为 D:\jekyll\Ruby200-x64，设置到环境变量中，设置完成后ruby -v检查安装是否成功；

2、安装DevKit-tdm-32-4.5.2-20110712-1620-sfx到D:\jekyll\devkit，
cd D:\jekyll\devkit
ruby dk.rb init 完成初始化配置
在配置文件config.yml最后一行添加 - D:/jekyll/Ruby200-x64
执行如下命令完成安装
ruby dk.rb review
ruby dk.rb install

3、检查gem是否正常 gem -v

4、安装jekyll执行命令gem install jekyll
若出现以下错误
ERROR:  Could not find a valid gem 'jekyll' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - Errno::ECONNRESET: An existing connection was forcibly closed 
by the remote host. - SSL_connect (https://rubygems.org/latest_specs.4.8.gz)
解决方案:
gem sources -r https://rubygems.org
gem sources -a http://rubygems.org
意思是将https替换成http，再gem install jekyll完成安装，安装完成后修改回原样
gem sources -r http://rubygems.org
gem sources -a https://rubygems.org

漫长的等待安装成功后将出现 14 gems installed 安装成功
使用命令 jekyll -v可查看版本

5、使用命令
jekyll new blog 完成新建
cd blog
jekyll build 编译
jekyll serve 启动服务 http://localhost:4000/可查看结果

若使用了插件(_config.yml中配置)则需先安装插件后才能正常编译，如安装如下插件(漫长的等待安装过程)
{% highlight ruby %}
gem install jekyll-paginate 
gem install jekyll-gist
{% endhighlight %}

结束，开始使用jekyll编写自己的博客吧！
