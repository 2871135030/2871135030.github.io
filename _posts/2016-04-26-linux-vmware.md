---
layout: post
title: Vmware linux拷贝网络故障
date:   2016-04-26 10:27:48
categories: [linux]
---

## Vmware linux拷贝Bringing up interface eth0: Device eth0 does not seem to be present ,delaying initialization

Wmware通过拷贝的原系统，重启网络(service network restart)会出现错误Bringing up interface eth0: Device eth0 does not seem to be present ,delaying initialization
错误截图如下
![Bringing up interface eth0](/assets/dfeeded7-8f45-3579-9195-e0e6933a8ad1.png)

<font color="red">分析原因：</font>

查看网络配置

```markdown
cat /etc/sysconfig/network-scripts/ifcfg-eth0
``` 

![Bringing up interface eth0](/assets/dfeeded7-8f45-3579-9195-e0e6933a8ad3.png)

```markdown
cat /etc/udev/rules.d/70-persistent-net.rules
``` 

![Bringing up interface eth0](/assets/dfeeded7-8f45-3579-9195-e0e6933a8ad2.png)

出现此故障的原因为虚拟系统安装时的网卡与迁移到新的虚拟机上的虚拟系统不一致，
当迁移到新虚拟机时重新配置了网上为eth1，但eth0已经失效，所以无法正确配置网络，
当然可以先删除 /etc/udev/rules.d/70-persistent-net.rules 文件，然后 reboot 重启配置网络

<font color="red">解决方法：</font>

将ifcfg-eth0配置中HWADDR改为70-persistent-net.rules配置中的eth1物理地址一致，
注意eth需保持一致。因为eth0已失效，所以将eth0

因为eth0已失效，所以将配置70-persistent-net.rules文件中eth0项删除

配置完成后 reboot 重启，网络即可生效，

配置结果如下：
![Bringing up interface eth0](/assets/dfeeded7-8f45-3579-9195-e0e6933a8ad4.png)

结束。
