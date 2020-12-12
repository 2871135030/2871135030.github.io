---
layout: post
title: Kubernetes-v1.19.3通过修改源码的方式完成CA根证书及普通证书续期（200年）
date:   2021-11-12 14:15:15
categories: [kubernetes]
---

#### Kubernetes-v1.19.3通过修改源码的方式完成CA根证书及普通证书续期（200年）

##### 1、检查原有集群的证书情况

* 检查集群证书情况

```
[root@kubernetes-master ~]# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Dec 03, 2021 15:25 UTC   364d                                    no      
apiserver                  Dec 03, 2021 15:25 UTC   364d            ca                      no      
apiserver-etcd-client      Dec 03, 2021 15:25 UTC   364d            etcd-ca                 no      
apiserver-kubelet-client   Dec 03, 2021 15:25 UTC   364d            ca                      no      
controller-manager.conf    Dec 03, 2021 15:25 UTC   364d                                    no      
etcd-healthcheck-client    Dec 03, 2021 15:25 UTC   364d            etcd-ca                 no      
etcd-peer                  Dec 03, 2021 15:25 UTC   364d            etcd-ca                 no      
etcd-server                Dec 03, 2021 15:25 UTC   364d            etcd-ca                 no      
front-proxy-client         Dec 03, 2021 15:25 UTC   364d            front-proxy-ca          no      
scheduler.conf             Dec 03, 2021 15:25 UTC   364d                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Dec 01, 2030 15:25 UTC   9y              no      
etcd-ca                 Dec 01, 2030 15:25 UTC   9y              no      
front-proxy-ca          Dec 01, 2030 15:25 UTC   9y              no      
[root@kubernetes-master ~]# 

```

* 重新生成证书`kubeadm alpha certs renew all`

```
[root@kubernetes-master ~]# kubeadm alpha certs renew all
[renew] Reading configuration from the cluster...
[renew] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

certificate embedded in the kubeconfig file for the admin to use and for kubeadm itself renewed
certificate for serving the Kubernetes API renewed
certificate the apiserver uses to access etcd renewed
certificate for the API server to connect to kubelet renewed
certificate embedded in the kubeconfig file for the controller manager to use renewed
certificate for liveness probes to healthcheck etcd renewed
certificate for etcd nodes to communicate with each other renewed
certificate for serving etcd renewed
certificate for the front proxy client renewed
certificate embedded in the kubeconfig file for the scheduler manager to use renewed
[root@kubernetes-master ~]# 
```

> 使用命令`kubeadm alpha certs check-expiration`重新生成证书后，可以看到除了`ca`、`etcd-ca`、`front-proxy-ca`三个ca证书未更新外，其余证书又延长到了一年后；下面开始编译源码并更改重新生成证书的期限；

##### 2、安装golang

* 下载`golang`安装包

```
https://golang.google.cn/dl/  国内使用此地址下载安装包 go1.15.5.linux-amd64.tar.gz
```

* 安装`golang`

```
解压到指定目录
tar -zxf go1.13.5.linux-amd64.tar.gz -C /usr/local

配置环境变量(vi /etc/profile)
export GO111MODULE=on
export GOROOT=/usr/local/go 
export PATH=$PATH:$GOROOT/bin

使环境变量生效
source /etc/profile
```

* 检查安装

```
[root@localhost local]# go version
go version go1.15.5 linux/amd64
[root@localhost local]# 
```


##### 3、编译kubernetes

* 下载kubernetes-v1.19.3源码包

```
从网址 https://github.com/kubernetes/kubernetes/releases 找到 v1.19.3源码包并下载（约32M），下载地址 https://codeload.github.com/kubernetes/kubernetes/tar.gz/v1.19.3
```

* 解压

```
cd /project/
tar -zxvf v1.19.3.tar.gz
cd kubernetes-1.19.3
```

* 修改ca根证书期限

```
编辑文件
vi ./staging/src/k8s.io/client-go/util/cert/cert.go
将now.Add(duration365d * 10).UTC()中的10年（默认）改为200年
```

* 修改普通证书期限

```
编辑文件
vi ./cmd/kubeadm/app/constants/constants.go
将CertificateValidity参数增加乘200年（默认1年）
```

* 编译

```
yum install gcc make -y
yum install rsync jq -y

此处只编译kubeadm即可，编译后的目录在_output/local/bin/linux/amd64/kubeadm（可另外编译kubelet或kubectl）
make all WHAT=cmd/kubeadm GOFLAGS=-v

```

##### 4、使用新编译的命令完成证书续期

* 替换kubeadm

```
备份原kubeadm命令后替换
[root@kubernetes-master ~]# which kubeadm
/usr/bin/kubeadm
[root@kubernetes-master ~]# mv /usr/bin/kubeadm /usr/bin/kubeadm_bk
[root@kubernetes-master ~]# chmod +x /usr/bin/kubeadm

```

> 替换后可使用命令`kubeadm alpha certs renew all`续期证书，但无法更改ca证书，集群需重建才可以；


* 重新生成证书并检查

```
[root@kubernetes-master bin]# kubeadm alpha certs check-expiration
[check-expiration] Reading configuration from the cluster...
[check-expiration] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'

CERTIFICATE                EXPIRES                  RESIDUAL TIME   CERTIFICATE AUTHORITY   EXTERNALLY MANAGED
admin.conf                 Oct 16, 2220 16:31 UTC   199y                                    no      
apiserver                  Oct 16, 2220 16:31 UTC   199y            ca                      no      
apiserver-etcd-client      Oct 16, 2220 16:31 UTC   199y            etcd-ca                 no      
apiserver-kubelet-client   Oct 16, 2220 16:31 UTC   199y            ca                      no      
controller-manager.conf    Oct 16, 2220 16:31 UTC   199y                                    no      
etcd-healthcheck-client    Oct 16, 2220 16:31 UTC   199y            etcd-ca                 no      
etcd-peer                  Oct 16, 2220 16:31 UTC   199y            etcd-ca                 no      
etcd-server                Oct 16, 2220 16:31 UTC   199y            etcd-ca                 no      
front-proxy-client         Oct 16, 2220 16:31 UTC   199y            front-proxy-ca          no      
scheduler.conf             Oct 16, 2220 16:31 UTC   199y                                    no      

CERTIFICATE AUTHORITY   EXPIRES                  RESIDUAL TIME   EXTERNALLY MANAGED
ca                      Oct 16, 2220 16:31 UTC   199y            no      
etcd-ca                 Oct 16, 2220 16:31 UTC   199y            no      
front-proxy-ca          Oct 16, 2220 16:31 UTC   199y            no      
[root@kubernetes-master bin]# 
[root@kubernetes-master bin]# 
[root@kubernetes-master bin]# 
```

> ???后续需继续研究如果在已经运行的集群中，并且保证不中断业务的情况下续期ca及其他相关证书





#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

