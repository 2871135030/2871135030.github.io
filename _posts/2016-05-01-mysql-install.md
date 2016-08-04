---
layout: post
title: mysql windows绿色版使用，导入导出功能
date:   2016-05-01 11:45:01
categories: [mysql,tools]
---

## mysql windows绿色版使用，导入导出功能

mysql windows绿色版安装步骤如下（下载mysql免安装包mysql-5.7.9-winx64.zip）：

1、将文件解压到目录D:\Program Files\mysql-5.7.9-winx64

2、copy目录下my-default.ini为my.ini，并修改其中2项如下

```markdown
basedir=D:\Program Files\mysql-5.7.9-winx64
datadir=D:\Program Files\mysql-5.7.9-winx64\data
```

3、cmd进入到D:\Program Files\mysql-5.7.9-winx64\bin，执行命令

```markdown
mysqld.exe --initialize-insecure --user=mysql 回车

mysqld install 回车
```

即可见到显示安装服务成功

4、通过命令 net start mysql 启动mysql数据库

5、通过命令 mysql -u root -p 登入（默认为无密码）进行修改密码


```markdown
mysqladmin -u root password "root123"
```

6、mysql的导入导出命令

```markdown
导出整个testdb数据库
mysqldump -h localhost testdb -uroot -proot > D:\dbbackup\testdb.dump
导入整个testdb（需先创建testdb数据库）
mysql -h localhost testdb -uroot -proot < D:\dbbackup\testdb.dump
```

mysql导入与导出命令同时可只操作单张表或多张表，详细操作说明请查看命令mysqldump mysql。

结束。