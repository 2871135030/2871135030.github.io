---
layout: post
title: maven安装第三方包到本地或私服
date:   2016-04-30 11:26:46
categories: [maven,tools]
---

## maven安装第三方包到本地或私服

如需安装到本地，确认本地maven环境正常可使用，可直接install，命令如下：

```markdown
mvn install:install-file -DgroupId=com.hode -DartifactId=quest -Dversion=1.0 -Dpackaging=jar -Dfile=quest.jar
```

说明：可以看到，上面的命令是将quest.jar包直接安装到本地的maven仓库中。

若需要安装到私服上，首先需配置用户的访问权限，在maven的配置文件setting.xml中配置thirdparty项，配置如下

```markdown
<server>
	<id>thirdparty</id>
	<username>root</username>
	<password>root</password>
</server>
```

安装命令如下:

```markdown
mvn deploy:deploy-file -DgroupId=com.hode -DartifactId=quest -Dversion=1.0 -Dpackaging=jar -Dfile=quest.jar -DrepositoryId=thirdparty -Durl=http://192.168.23.57:8081/nexus/content/repositories/thirdparty/
```


结束。