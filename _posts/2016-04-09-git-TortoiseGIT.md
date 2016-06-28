---
layout: post
title: TortoiseGIT记住用户名密码简单方法
date:   2016-04-09 13:31:11
categories: tools
---

## TortoiseGIT记住用户名密码简单方法

我们在使用TortoiseGIT时，每次将代码push到GIT服务器上时都需用户名密码，浪费不少时间，需要记住用户名密码只需找到当前用户目录(C:\Users\administrator\)下的文件.gitconfig
增加如下配置项即可(请注意空格)

{% highlight ruby %}
[credential]
	helper = store
{% endhighlight %}

配置完成后，首次push操作录入用户名密码，此后push操作均无需密码了，结束。
