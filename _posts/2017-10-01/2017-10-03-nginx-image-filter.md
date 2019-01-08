---
layout: post
title: Nginx使用image-filter模块生成缩略图
date:   2017-10-03 11:31:58
categories: [tools]
---

## Nginx使用image-filter模块生成缩略图

1、nginx安装步骤可参考<a href="/linux/tools/2016/05/12/nginx-linux-install.html">Linux环境下安装nginx</a>，默认安装是没有image-filter模块的，我们需要动态增加此模块；
{% highlight ruby %}
先安装好image-filter模块所需的依赖
yum install gd-devel -y
到nginx的目录执行命令完成配置
./configure --prefix=/usr/local/nginx-1.14.2 --conf-path=/usr/local/nginx-1.14.2/nginx.conf --with-http_ssl_module --with-http_stub_status_module --with-http_image_filter_module  --with-pcre
配置成功后执行编译(注意不需要执行安装 make install)
make
编译成功后将文件覆盖到安装目录下
cp objs/nginx /usr/local/nginx-1.14.2/sbin/
使用以下命令可查看应用的模块
./nginx -V
{% endhighlight %}

2、配置nginx.conf，在server中增加如下配置完成图片缩放处理
{% highlight ruby %}
    location ~* /(.*)_(\d+)x(\d+)\.(jpg|gif|png)$ {
        root html;
        set $name $1;
        set $w $2;
        set $h $3;
		set $type $4;
        image_filter resize $w $h;
        image_filter_buffer 10M;
        try_files /$name.$type /404.jpg;
    }
{% endhighlight %}

3、访问图片，在html目录中上传图片测试
{% highlight ruby %}
原图
http://172.16.8.103/20190108234809.jpg
缩略图
http://172.16.8.103/20190108234809_400x400.jpg
{% endhighlight %}
结束。
