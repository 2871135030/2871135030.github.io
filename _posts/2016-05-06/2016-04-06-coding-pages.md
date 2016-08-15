---
layout: post
title: Coding pages + Github pages 搭建自己的博客
date:   2016-04-06 10:13:56
categories: others
---

## Coding pages + Github pages 搭建自己的博客

本文以博主实际申请操作为例，详述通过申请域名及免费空间Github pages,Coding pages(Coding pages主要是为了解决国内用户访问，百度不收录的问题) 搭建博客。

1、申请个域名<a href="http://www.mixfate.com">www.mixfate.com</a>，为了免去备案的麻烦，选择申请国际域名(.com)。

2、在github创建一个与用户名相同的项目，资源命名必须符合这样的规则username/username.github.io，右边菜单中的Settings按钮，在跳转到的页面 Update your site 对应处点击“Automatic page generator”按钮，这样就有了一个github自动生成的页面用来测试的时候使用。之后点击继续。
来到选择主题界面，选择主题并发布。再次点击右侧“Settings”按钮，在页面中点击博客地址链接（或者直接在浏览器输入http://username.github.io）即可看到自己当前的博客首页了（如果是第一次点击可能会出现404这时候需要等十分钟之后就可以）

3、创建完成后就可以使用Git客户端更新代码了，本地使用jekyll完成博客发布后将重新push到github。
绑定域名设置需在项目的根目录创建CNAME文件，内容为www.mixfate.com并上传到github

4、在域名解析中配置解析到github，如图所示
![Coding pages + Github pages](/assets/dfeeded7-8f45-3579-9195-e0e6933a8adb.png)
此处配置了海外直接访问github，国内还是访问Coding pages不受限制

5、在Coding中创建一个项目，并创建一个分支命名为coding-pages，同时将此分支设置成默认，将Github代码全量上传一份。在Coding Pages 服务中选择绑定www.mixfate.com，绑定域名后域名解析配置CNAME 记录指向 pages.coding.me 即可访问。

接下来就观察百度，谷歌的收录情况了。

持续更新中。。。

end.


