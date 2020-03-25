---
layout: post
title: Elastic Search常用操作整理
date:   2017-10-07 13:22:58
categories: [spring]
---

## Elastic Search常用操作整理

1、准备好机器，本文演示使用vmware虚拟出3台机器(三台机器均需操作本步骤)
{% highlight ruby %}
#修改/etc/hosts文件
cat >> /etc/hosts << EOF
192.168.26.160 k8s-master
192.168.26.161 k8s-node1
192.168.26.162 k8s-node2
EOF

#关闭防火墙和selinux
systemctl stop firewalld && systemctl disable firewalld
sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config && setenforce 0

#关闭swap
swapoff -a
yes | cp /etc/fstab /etc/fstab_bak
cat /etc/fstab_bak |grep -v swap > /etc/fstab
{% endhighlight %}

修改iptables相关参数
{% highlight ruby %}
cat <<EOF >  /etc/sysctl.d/k8s.conf
vm.swappiness = 0
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# 使配置生效
modprobe br_netfilter
sysctl -p /etc/sysctl.d/k8s.conf
{% endhighlight %}
加载ipvs相关模块
{% highlight ruby %}
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

#执行脚本
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
{% endhighlight %}

安装docker(最好使用yumdownloader下载离线安装)，离线包如下
{% highlight ruby %}
audit-libs-python-2.8.4-4.el7.x86_64.rpm
checkpolicy-2.5-8.el7.x86_64.rpm
container-selinux-2.74-1.el7.noarch.rpm
docker-ce-18.06.1.ce-3.el7.x86_64.rpm
docker-ce-18.06.1.tar.gz
libcgroup-0.41-20.el7.x86_64.rpm
libsemanage-python-2.5-14.el7.x86_64.rpm
libtool-ltdl-2.4.2-22.el7_3.x86_64.rpm
policycoreutils-python-2.5-29.el7.x86_64.rpm
python-IPy-0.75-6.el7.noarch.rpm
setools-libs-3.3.8-4.el7.x86_64.rpm
#执行安装命令完成安装
[root@k8s-master software]# rpm -i *.rpm --force
[root@k8s-master software]# docker -v
Docker version 18.06.1-ce, build e68fc7a
{% endhighlight %}


2、k8s-master机器安装
{% highlight ruby %}
#导入以下镜像(镜像来源)
docker tag ff281650a721 quay.io/coreos/flannel:v0.11.0-amd64
docker tag fdb321fd30a0 registry.aliyuncs.com/google_containers/kube-proxy:v1.13.1
docker tag 26e6f1db2a52 registry.aliyuncs.com/google_containers/kube-controller-manager:v1.13.1
docker tag 40a63db91ef8 registry.aliyuncs.com/google_containers/kube-apiserver:v1.13.1
docker tag ab81d7360408 registry.aliyuncs.com/google_containers/kube-scheduler:v1.13.1
docker tag f59dcacceff4 registry.aliyuncs.com/google_containers/coredns:1.2.6 
docker tag 3cab8e1b9802 registry.aliyuncs.com/google_containers/etcd:3.2.24
docker tag da86e6ba6ca1 registry.aliyuncs.com/google_containers/pause:3.1
#安装软件
[root@k8s-master software]# ll
total 51260
-rw-r--r--. 1 root root 21541054 Dec 14 11:25 25cd948f63fea40e81e43fbe2e5b635227cc5bbda6d5e15d42ab52decf09a5ac-kubelet-1.13.1-0.x86_64.rpm
-rw-r--r--. 1 root root  4409594 Oct  8 12:05 53edc739a0e51a4c17794de26b13ee5df939bd3161b37f503fe2af8980b41a89-cri-tools-1.12.0-0.x86_64.rpm
-rw-r--r--. 1 root root  8313914 Dec 14 11:24 5af5ecd0bc46fca6c51cc23280f0c0b1522719c282e23a2b1c39b8e720195763-kubeadm-1.13.1-0.x86_64.rpm
-rw-r--r--. 1 root root  8907502 Dec 14 11:24 7855313ff2b42ebcf499bc195f51d56b8372abee1a19bbf15bb4165941c0229d-kubectl-1.13.1-0.x86_64.rpm
-rw-r--r--. 1 root root  9008838 Mar  5  2018 fe33057ffe95bfae65e2f269e1b05e99308853176e24a4d027bc082b471a07c0-kubernetes-cni-0.6.0-0.x86_64.rpm
-rw-r--r--. 1 root root   296632 Aug 10  2017 socat-1.7.3.2-2.el7.x86_64.rpm
[root@k8s-master software]# rpm -i *.rpm --force
[root@k8s-master software]# systemctl enable kubelet && systemctl start kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /etc/systemd/system/kubelet.service.
[root@k8s-master software]# 
#初始化
kubeadm init \
    --apiserver-advertise-address=192.168.26.160 \
    --image-repository registry.aliyuncs.com/google_containers \
    --kubernetes-version v1.13.1 \
    --pod-network-cidr=10.244.0.0/16
#按提示执行以下命令
  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
#节点加入集群命令如下
  kubeadm join 192.168.26.160:6443 --token iswczf.3q2t4x9nevairfn2 --discovery-token-ca-cert-hash sha256:0de6096281d1daa66a12ba2dceb7e8687578dd227cdc34f9aba9214ded619eff
#部署网络插件
获取配置文件
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 
kubectl apply -f kube-flannel.yml
{% endhighlight %}

3、k8s-node1\k8s-node2安装
{% highlight ruby %}
#安装软件
[root@k8s-master software]# ll
total 51260
-rw-r--r--. 1 root root 21541054 Dec 14 11:25 25cd948f63fea40e81e43fbe2e5b635227cc5bbda6d5e15d42ab52decf09a5ac-kubelet-1.13.1-0.x86_64.rpm
-rw-r--r--. 1 root root  4409594 Oct  8 12:05 53edc739a0e51a4c17794de26b13ee5df939bd3161b37f503fe2af8980b41a89-cri-tools-1.12.0-0.x86_64.rpm
-rw-r--r--. 1 root root  8313914 Dec 14 11:24 5af5ecd0bc46fca6c51cc23280f0c0b1522719c282e23a2b1c39b8e720195763-kubeadm-1.13.1-0.x86_64.rpm
-rw-r--r--. 1 root root  8907502 Dec 14 11:24 7855313ff2b42ebcf499bc195f51d56b8372abee1a19bbf15bb4165941c0229d-kubectl-1.13.1-0.x86_64.rpm
-rw-r--r--. 1 root root  9008838 Mar  5  2018 fe33057ffe95bfae65e2f269e1b05e99308853176e24a4d027bc082b471a07c0-kubernetes-cni-0.6.0-0.x86_64.rpm
-rw-r--r--. 1 root root   296632 Aug 10  2017 socat-1.7.3.2-2.el7.x86_64.rpm
[root@k8s-master software]# rpm -i *.rpm --force
[root@k8s-master software]# systemctl enable kubelet && systemctl start kubelet
Created symlink from /etc/systemd/system/multi-user.target.wants/kubelet.service to /etc/systemd/system/kubelet.service.
[root@k8s-master software]# 
#导入以下镜像(镜像来源)
docker tag ff281650a721 quay.io/coreos/flannel:v0.11.0-amd64
docker tag fdb321fd30a0 registry.aliyuncs.com/google_containers/kube-proxy:v1.13.1
docker tag da86e6ba6ca1 registry.aliyuncs.com/google_containers/pause:3.1
#执行加入集群命令
kubeadm join 192.168.26.160:6443 --token iswczf.3q2t4x9nevairfn2 --discovery-token-ca-cert-hash sha256:0de6096281d1daa66a12ba2dceb7e8687578dd227cdc34f9aba9214ded619eff

{% endhighlight %}




5、master上检查
{% highlight ruby %}
[[root@k8s-master software]# kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   34m   v1.13.1
k8s-node1    Ready    <none>   12m   v1.13.1
k8s-node2    Ready    <none>   11m   v1.13.1
[root@k8s-master software]#
{% endhighlight %}

结束。
