---
layout: post
title: centos minimal使用ssh/scp命令不录入密码
date:   2016-05-05 12:09:48
categories: [linux]
---

## centos minimal使用ssh/scp命令不录入密码

不同的Linux服务器之间经常需要文档拷贝，本例演示在已安装的两台CentOS-6.7-x86_64-minimal机器上使用scp传输文件。

暂定义机器A(ip:192.168.88.102)，机器B(ip:192.168.88.103)，

在机器B上操作如下命令

```markdown
scp -p /software/download/quartz-2.2.1-distribution.tar.gz root@192.168.88.102:/software/download/quartz-2.2.1-distribution.tar.gz
```

表示将机器B上的文件拷贝到机上A上，未做任何配置时，需输入root的密码，日志如下

```markdown
[root@centos-b ~]# scp -p /software/download/quartz-2.2.1-distribution.tar.gz root@192.168.88.102:/software/download/quartz-2.2.1-distribution.tar.gz
The authenticity of host '192.168.88.102 (192.168.88.102)' can't be established.
RSA key fingerprint is dd:b1:b3:ca:5d:17:77:0e:20:44:f6:19:c5:5e:a7:9c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.88.102' (RSA) to the list of known hosts.
root@192.168.88.102's password: 
quartz-2.2.1-distribution.tar.gz                                      100% 3216KB   3.1MB/s   00:00    
[root@centos-b ~]# 
```

可以看到每次都输入密码，为了简化操作，创建key实现两台机器自由传输文件。

在机器B上执行ssh-keygen -t rsa，如下

```markdown
[root@centos-b ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
c9:c4:96:3f:c0:da:5e:e7:58:e5:49:91:ba:bb:81:c1 root@centos-b
The key's randomart image is:
+--[ RSA 2048]----+
|              .. |
|       o .    .. |
|        B    .o  |
|       * =  .+ . |
|      . S E o.o  |
|       . . O.    |
|        . o o.   |
|            ..   |
|            ..   |
+-----------------+
[root@centos-b ~]# 
```

可以看到生成了机器的公钥和私钥，分别为目录/root/.ssh下的id_rsa.pub及id_rsa。

将B机器生成的公钥拷贝到A机器的指定目录中（若无此目录，则直接创建），在B机器上执行

```markdown
scp -p /root/.ssh/id_rsa.pub root@192.168.88.102:/root/.ssh/id_rsa103.pub
```

拷贝完成后，可看到A机器目录/root/.ssh/中已有文件id_rsa103.pub

接下来在A机器上执行

```markdown
cat id_rsa103.pub >> authorized_keys
```

文件拷贝完成，编辑文件/etc/ssh/sshd_config

```markdown
vi /etc/ssh/sshd_config

设置以下两项（如已注释，则取消注释）
PubkeyAuthentication yes
AuthorizedKeysFile      .ssh/authorized_keys
```

最后使用restorecon恢复属性，并重启ssh服务即可

```markdown
restorecon -Rv ~/.ssh
service sshd restart
```

注：非root权限可能存在文件权限问题，需要配置权限，如下：

```markdown
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys 
```

结束。
