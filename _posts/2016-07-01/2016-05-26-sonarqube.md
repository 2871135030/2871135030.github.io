---
layout: post
title: SonarQube 6.2代码质量管理平台win7安装与使用
date:   2016-05-26 12:59:58
categories: [tools]
---

## SonarQube 6.2代码质量管理平台win7安装与使用

1、官网下载sonarqube-6.2.zip、sonar-scanner-2.8.zip(请忽略文章发布日期)，并分别解压，
如解压后目录为F:\newtool\sonar\sonarqube-6.2、F:\newtool\sonar\sonar-scanner-2.8。

2、安装好mysql，并创建一个数据库sonar，修改配置文件F:\newtool\sonar\sonarqube-6.2\conf\sonar.properties，增加或修改以下配置

```markdown
sonar.jdbc.username=root
sonar.jdbc.password=kroot
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
```

3、进入目录F:\newtool\sonar\sonarqube-6.2\bin\windows-x86-64，运行StartSonar.bat启动sonarqube，若需要关闭请键一次Ctrl+C，等待jvm进程关闭后，然后输入Y停止。

4、启动成功后，打开http://localhost:9000/，右上角点击Login，默认用户名及密码均为admin。

5、登录成功后，点击顶部Administration，然后选择System->Update Center，再选择Available标签，查找SonarQube Chinese Pack并install安装中文插件。
安装完成后重启sonar，即可看到中文插件已生效，同时可以看到F:\newtool\sonar\sonarqube-6.2\extensions\plugins新增了sonar-l10n-zh-plugin-1.13.jar插件包。

6、打开sonar scanner配置文件F:\newtool\sonar\sonar-scanner-2.8\conf\sonar-scanner.properties，增加或修改以下配置项即可

```markdown
sonar.host.url=http://localhost:9000
sonar.sourceEncoding=UTF-8
```

7、确保sonarqube已启动，命令行进入待管理的代码根目录(可包括子文件夹中的项目)，执行以下命令

```markdown
F:\newtool\sonar\sonar-scanner-2.8\bin\sonar-scanner  -Dsonar.projectKey=sonar:testproject -Dsonar.sources=.
```

扫描成功后，即可用http://localhost:9000/查看结果。

8、sonar同样提供了maven插件，可直接使用插件完成。


结束。
