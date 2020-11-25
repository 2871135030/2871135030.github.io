---
layout: post
title: Kubernetes-v1.19.3集群使用kubeadm安装
date:   2021-11-12 14:15:13
categories: [java]
---

#### Kubernetes-v1.19.3集群使用kubeadm安装

##### 1、基础环境准备
###### 1.1、机器说明，共三台机器一台master，两台node节点

|机器说明|ip|系统|
|-|-|-|
|master|192.168.80.10|centos7.8|
|node1|192.168.80.20|centos7.8|
|node2|192.168.80.30|centos7.8|

```
[root@kubernetes-master ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)
[root@kubernetes-master ~]# rpm -q kernel
kernel-3.10.0-1127.19.1.el7.x86_64
[root@kubernetes-master ~]# 
```
###### 1.2、三台机器均配置host
```
192.168.80.10 kubernetes-master
192.168.80.20 kubernetes-node20
192.168.80.30 kubernetes-node30
```
###### 1.3、三台机器均操作以下配置
* 关闭`SELINUX`

```
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```
* 关闭`swap`

```
swapoff -a
sed -i 's/^[^#].*swap/#&/' /etc/fstab
systemctl daemon-reload
```

* 关闭`ipv6`

```
echo net.ipv6.conf.all.disable_ipv6=1 >> /etc/sysctl.conf
echo NETWORKING_IPV6=no >> /etc/sysconfig/network
sed -i 's/IPV6INIT=yes/IPV6INIT=no/g' /etc/sysconfig/network-scripts/ifcfg-ens33
sysctl -p
ip a 查看ipv6是否关闭
```
* 关闭防火墙

```
systemctl stop firewalld
systemctl disable firewalld
systemctl status firewalld
```
* 配置将桥接流量传递到iptables并执行生效

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sudo sysctl --system
```

###### 1.4、三台机器均安装docker
* 卸载可能存在的旧版docker

```
sudo yum remove docker \
                   docker-client \
                   docker-client-latest \
                   docker-common \
                   docker-latest \
                   docker-latest-logrotate \
                   docker-logrotate \
                   docker-engine
```
* 下载最新版本docker的rpm安装包进行本地安装

[docker官网下载最新的安装包](https://download.docker.com/linux/centos/7/x86_64/stable/Packages/)
> 安装包如下

```
containerd.io-1.3.7-3.1.el7.x86_64.rpm
docker-ce-19.03.13-3.el7.x86_64.rpm
docker-ce-cli-19.03.13-3.el7.x86_64.rpm
```
> 本地安装docker

```
yum localinstall *.rpm -y --nogpgcheck
```
> 设置开机启动并启动docker

```
systemctl enable docker
systemctl start docker
```
> 修改docker的cgroupdriver为systemd（默认为cgroups），因kubernetes默认使用的cgroupdriver为systemd（kubernetes自带的推荐使用）

```
vi /etc/docker/daemon.json

{
    "exec-opts":["native.cgroupdriver=systemd"]
}

systemctl restart docker
```
###### 1.5、master机器设置免登录node20、node30（在master上执行以下命令）
```
ssh-keygen -t rsa (一路默认回车即可)
ssh-copy-id kubernetes-master
ssh-copy-id kubernetes-node20
ssh-copy-id kubernetes-node30
```

###### 1.6、镜像准备
* 预先从阿里云镜像获取到对应版本的镜像

```
docker pull registry.aliyuncs.com/google_containers/kube-apiserver:v1.19.3
docker pull registry.aliyuncs.com/google_containers/kube-controller-manager:v1.19.3
docker pull registry.aliyuncs.com/google_containers/kube-scheduler:v1.19.3
docker pull registry.aliyuncs.com/google_containers/kube-proxy:v1.19.3
docker pull registry.aliyuncs.com/google_containers/pause:3.2
docker pull registry.aliyuncs.com/google_containers/etcd:3.4.13-0
docker pull registry.aliyuncs.com/google_containers/coredns:1.7.0

拿下来后统一改名（master节点需要以下所有镜像）

docker tag registry.aliyuncs.com/google_containers/kube-apiserver:v1.19.3           k8s.gcr.io/kube-apiserver:v1.19.3
docker tag registry.aliyuncs.com/google_containers/kube-controller-manager:v1.19.3  k8s.gcr.io/kube-controller-manager:v1.19.3
docker tag registry.aliyuncs.com/google_containers/kube-scheduler:v1.19.3           k8s.gcr.io/kube-scheduler:v1.19.3
docker tag registry.aliyuncs.com/google_containers/kube-proxy:v1.19.3               k8s.gcr.io/kube-proxy:v1.19.3
docker tag registry.aliyuncs.com/google_containers/pause:3.2                        k8s.gcr.io/pause:3.2
docker tag registry.aliyuncs.com/google_containers/etcd:3.4.13-0                    k8s.gcr.io/etcd:3.4.13-0
docker tag registry.aliyuncs.com/google_containers/coredns:1.7.0                    k8s.gcr.io/coredns:1.7.0
```
* 另使用calico网络插件，所需依赖的镜像国内网络环境可自动下载

```
calico/node:v3.16.5
calico/pod2daemon-flexvol:v3.16.5
calico/cni:v3.16.5
calico/kube-controllers:v3.16.5
```
###### 1.7、 kubernetes安装包下载
[安装包阿里云镜像下载](http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/Packages/)
* 下载的安装包如下

```
14bfe6e75a9efc8eca3f638eb22c7e2ce759c67f95b43b16fae4ebabde1549f3-cri-tools-1.13.0-0.x86_64.rpm
1de80331b548f69c71f62d733ad957e4153e9c9af5528d3dea74a3088c0b8421-kubelet-1.19.3-0.x86_64.rpm
24a0b394551c612fdc827c5f46a10bd1472fd79d070c2d0310c06369ce97f23a-kubeadm-1.19.3-0.x86_64.rpm
d088f1c423aca204ae08d0da14b244da33897ec16198d7aae117da2caf0b3141-kubectl-1.19.3-0.x86_64.rpm
db7cb5cb0b3f6875f54d10f02e625573988e3e91fd4fc5eef0b1876bb18604ad-kubernetes-cni-0.8.7-0.x86_64.rpm
```


##### 2、master安装
###### 2.1、 导入镜像(此tar镜像为单独下载并导出以作离线安装使用)

> 可使用`kubeadm config images list --kubernetes-version=1.19.3`检查安装所依赖的镜像

```
docker load < calico+cni:v3.16.5.tar
docker load < calico+kube-controllers:v3.16.5.tar
docker load < calico+node:v3.16.5.tar
docker load < calico+pod2daemon-flexvol:v3.16.5.tar
docker load < k8s.gcr.io+coredns:1.7.0.tar
docker load < k8s.gcr.io+etcd:3.4.13-0.tar
docker load < k8s.gcr.io+kube-apiserver:v1.19.3.tar
docker load < k8s.gcr.io+kube-controller-manager:v1.19.3.tar
docker load < k8s.gcr.io+kube-proxy:v1.19.3.tar
docker load < k8s.gcr.io+kube-scheduler:v1.19.3.tar
docker load < k8s.gcr.io+pause:3.2.tar

docker tag c1fa37765208 calico/node:v3.16.5
docker tag 178cfd5d2400 calico/pod2daemon-flexvol:v3.16.5
docker tag 9165569ec236 calico/cni:v3.16.5
docker tag 1120bf0b8b41 calico/kube-controllers:v3.16.5
docker tag cdef7632a242 k8s.gcr.io/kube-proxy:v1.19.3
docker tag 9b60aca1d818 k8s.gcr.io/kube-controller-manager:v1.19.3
docker tag a301be0cd44b k8s.gcr.io/kube-apiserver:v1.19.3
docker tag aaefbfa906bd k8s.gcr.io/kube-scheduler:v1.19.3
docker tag 0369cf4303ff k8s.gcr.io/etcd:3.4.13-0
docker tag bfe3a36ebd25 k8s.gcr.io/coredns:1.7.0
docker tag 80d28bedfe5d k8s.gcr.io/pause:3.2   

```
###### 2.2、安装kubeadm kubectl kubelet

```
[root@kubernetes-master software]# ls *.rpm
14bfe6e75a9efc8eca3f638eb22c7e2ce759c67f95b43b16fae4ebabde1549f3-cri-tools-1.13.0-0.x86_64.rpm
1de80331b548f69c71f62d733ad957e4153e9c9af5528d3dea74a3088c0b8421-kubelet-1.19.3-0.x86_64.rpm
24a0b394551c612fdc827c5f46a10bd1472fd79d070c2d0310c06369ce97f23a-kubeadm-1.19.3-0.x86_64.rpm
d088f1c423aca204ae08d0da14b244da33897ec16198d7aae117da2caf0b3141-kubectl-1.19.3-0.x86_64.rpm
db7cb5cb0b3f6875f54d10f02e625573988e3e91fd4fc5eef0b1876bb18604ad-kubernetes-cni-0.8.7-0.x86_64.rpm
[root@kubernetes-master software]# yum localinstall *.rpm -y --nogpgcheck
```

###### 2.3、初始化master
```
kubeadm init --kubernetes-version=1.19.3
```

* 按提示初始化完执行命令

```
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
* 按提示打印出节点加入的方法

```
kubeadm join 192.168.80.10:6443 --token 945iyw.ejofmbm5mi1rbong \
    --discovery-token-ca-cert-hash sha256:2835ace5eed724587d0467a789a0b902a917f27959203a1a5e8428e38e125f2a
```
> 可通过`kubeadm token list`查看当前的token，默认有效期为24小时，若忘记token或token过期，可使用命令`kubeadm token create --print-join-command`重新生成，若初始有问题可通过命令`kubeadm reset`重置。

###### 2.4、启动kubelet并检查状态
```
systemctl start kubelet
systemctl status kubelet
```
> 若启动出错，可使用命令`journalctl -xefu kubelet`跟踪错误信息

> 出现`kubelet cgroup driver: "cgroupfs" is different from docker cgroup driver: "systemd"`需修改k8s配置文件指定cgroup为systemd，修改配置文件`/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf`，在`KUBELET_KUBECONFIG_ARGS`参数增加配置`--cgroup-driver=systemd`，修改配置文件`/var/lib/kubelet/kubeadm-flags.env`，在`KUBELET_KUBEADM_ARGS`参数增加配置`--cgroup-driver=systemd`，修改完后执行`systemctl daemon-reload ; systemctl restart docker kubelet`
###### 2.5、查看集群节点状态
```
[root@kubernetes-master ~]# kubectl get node
NAME                STATUS     ROLES    AGE   VERSION
kubernetes-master   NotReady   master   13m   v1.19.3
```
> 可以看到master节点是NotReady状态
###### 2.6、安装calico网络插件
[下载calico网络插件配置文件](https://docs.projectcalico.org/manifests/calico.yaml)
> 可使用命令` grep image calico.yaml`查看calico所需的镜像

* 安装calico命令

```
kubectl apply -f calico.yaml 
```

> 安装完成后即可看到master为Ready状态

```
[root@kubernetes-master ~]# kubectl get node
NAME                STATUS   ROLES    AGE   VERSION
kubernetes-master   Ready    master   19m   v1.19.3
```
###### 2.7、检查组件状态(`kubectl get cs`|`kubectl get componentstatus`)
```
[root@kubernetes-master ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused   
etcd-0               Healthy     {"health":"true"} 
```
> 可以看到scheduler与controller-manager是Unhealthy的，按端口提示检查10251及10252是否有监听

```
ss -ant|grep 10251
ss -ant|grep 10252
```
> 发现均未有监听，检查配置文件`/etc/kubernetes/manifests/kube-scheduler.yaml`与`/etc/kubernetes/manifests/kube-controller-manager.yaml`并将两个配置文件的配置项`- --port=0`均注释掉，并重启kubelet，重新检查组件状态

```
[root@kubernetes-master ~]# systemctl restart kubelet
[root@kubernetes-master ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
etcd-0               Healthy   {"health":"true"}   
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
[root@kubernetes-master ~]#
```


##### 3、node安装
###### 3.1 导入节点所需的镜像
```
docker load < calico+node:v3.16.5.tar
docker load < calico+pod2daemon-flexvol:v3.16.5.tar
docker load < calico+cni:v3.16.5.tar
docker load < calico+kube-controllers:v3.16.5.tar
docker load < k8s.gcr.io+kube-proxy:v1.19.3.tar
docker load < k8s.gcr.io+pause:3.2.tar

docker tag c1fa37765208  calico/node:v3.16.5
docker tag 178cfd5d2400  calico/pod2daemon-flexvol:v3.16.5
docker tag 9165569ec236  calico/cni:v3.16.5
docker tag 1120bf0b8b41  calico/kube-controllers:v3.16.5
docker tag cdef7632a242  k8s.gcr.io/kube-proxy:v1.19.3
docker tag 80d28bedfe5d  k8s.gcr.io/pause:3.2


```

###### 3.2 安装kubeadm kubectl kubelet
```
[root@kubernetes-master software]# ls *.rpm
14bfe6e75a9efc8eca3f638eb22c7e2ce759c67f95b43b16fae4ebabde1549f3-cri-tools-1.13.0-0.x86_64.rpm
1de80331b548f69c71f62d733ad957e4153e9c9af5528d3dea74a3088c0b8421-kubelet-1.19.3-0.x86_64.rpm
24a0b394551c612fdc827c5f46a10bd1472fd79d070c2d0310c06369ce97f23a-kubeadm-1.19.3-0.x86_64.rpm
d088f1c423aca204ae08d0da14b244da33897ec16198d7aae117da2caf0b3141-kubectl-1.19.3-0.x86_64.rpm
db7cb5cb0b3f6875f54d10f02e625573988e3e91fd4fc5eef0b1876bb18604ad-kubernetes-cni-0.8.7-0.x86_64.rpm
[root@kubernetes-master software]# yum localinstall *.rpm -y --nogpgcheck
```

###### 3.3 启动kubelet并加入集群
```
systemctl enable kubelet
systemctl start kubelet
kubeadm join 192.168.80.10:6443 --token yke737.yvu9ciyi09hdkscz     --discovery-token-ca-cert-hash sha256:2835ace5eed724587d0467a789a0b902a917f27959203a1a5e8428e38e125f2a 
```

##### 4、集群检查
###### 4.1、检查节点
```
[root@kubernetes-master ~]# kubectl get node
NAME                STATUS   ROLES    AGE     VERSION
kubernetes-master   Ready    master   49m     v1.19.3
kubernetes-node20   Ready    <none>   2m54s   v1.19.3
kubernetes-node30   Ready    <none>   2m50s   v1.19.3
[root@kubernetes-master ~]# 
```
###### 4.2、检查基础应用
```
[root@kubernetes-master ~]# kubectl get po -A
NAMESPACE     NAME                                        READY   STATUS    RESTARTS   AGE
kube-system   calico-kube-controllers-5c6f6b67db-lcctk    1/1     Running   0          31m
kube-system   calico-node-7n7j2                           1/1     Running   0          3m21s
kube-system   calico-node-g5kvd                           1/1     Running   0          31m
kube-system   calico-node-t47vg                           1/1     Running   0          3m17s
kube-system   coredns-f9fd979d6-pmht4                     1/1     Running   0          49m
kube-system   coredns-f9fd979d6-w4vcl                     1/1     Running   0          49m
kube-system   etcd-kubernetes-master                      1/1     Running   0          49m
kube-system   kube-apiserver-kubernetes-master            1/1     Running   0          49m
kube-system   kube-controller-manager-kubernetes-master   1/1     Running   0          21m
kube-system   kube-proxy-cz6hl                            1/1     Running   0          3m21s
kube-system   kube-proxy-h2d9g                            1/1     Running   0          49m
kube-system   kube-proxy-wxx5k                            1/1     Running   0          3m17s
kube-system   kube-scheduler-kubernetes-master            1/1     Running   0          21m
[root@kubernetes-master ~]# 
```



#### 赞赏(Donation)


##### 微信(Wechat Pay)

![donation-wechatpay](/assets/img/donate-wechatpay.png)

