---
title: kubeadm + vagrant 部署多节点 k8s 的一个坑
date: 2019-08-06 09:33:40
categories: Kubernetes
tags: Kubernetes
---

之前写过一篇[「使用 kubeadm 搭建 kubernetes 集群」](https://qhh.me/2019/03/19/%E4%BD%BF%E7%94%A8-kubeadm-%E6%90%AD%E5%BB%BA-kubernetes-%E9%9B%86%E7%BE%A4/)教程，教程里面使用 Vagrant 启动 3 个节点，1 个 master，2 个 node 节点，后来使用过程中才慢慢发现还是存在问题的。具体问题表现是：
1. kubectl get node -o wide 查看到节点 IP 都是：10.0.2.15；
```bash
[root@master ~]# kubectl get node -o wide
NAME     STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION              CONTAINER-RUNTIME
master   Ready    master   5m11s   v1.15.1   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
node1    Ready    <none>   2m31s   v1.15.1   10.0.2.15     <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
```
2. kubect get pod 可以查看 pod，pod 也运行正常，但是无法查看 pod 日志，也无法 kubectl exec -it 进入 pod。具体报错如下：
```bash
[root@master ~]# kubectl get pod
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5754944d6c-gnqjg   1/1     Running   0          66s
nginx-deployment-5754944d6c-mxgn7   1/1     Running   0          66s

[root@master ~]# kubectl logs nginx-deployment-5754944d6c-gnqjg
Error from server (NotFound): the server could not find the requested resource ( pods/log nginx-deployment-5754944d6c-gnqjg)

[root@master ~]# kubectl exec -it nginx-deployment-5754944d6c-gnqjg sh
error: unable to upgrade connection: pod does not exist
```


带着上面两个问题，于是网上搜索一番，找到了根因并得以解决：
https://github.com/kubernetes/kubernetes/issues/60835

#### 主要原因
Vagrant 在多主机模式时每个主机的 eth0 网口 ip 都是 10.0.2.15，这个网口是所有主机访问公网的出口，用于 nat 转发。而 eth1才是主机真正的 IP。kubelet 在启动时默认读取的是 eth0 网卡的 IP，因此在集群部署完后 kubect get node -o wide 查看到节点的 IP 都是 10.0.2.15。
```
[vagrant@master ~]$ ip addr
...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:c9:c7:04 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85708sec preferred_lft 85708sec
    inet6 fe80::5054:ff:fec9:c704/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:25:1b:45 brd ff:ff:ff:ff:ff:ff
    inet 192.168.26.10/24 brd 192.168.26.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe25:1b45/64 scope link
       valid_lft forever preferred_lft forever
...
```

#### 解决方法
上面知道问题的根本原因是 k8s 节点 IP 获取不对导致访问节点出现问题，那么解决方法就是调整 kubelet 参数设置正确的
IP 地址：
编辑 /etc/sysconfig/kubelet 文件，KUBELET_EXTRA_ARGS 环境变量添加 `--node-ip` 参数：
```
KUBELET_EXTRA_ARGS="--node-ip=<eth1 网口 IP>"
```
然后重启 kubelet：systemctl restart kubelet
执行 kubectl get node -o wide 发现节点 IP 已经改变成了KUBELET_EXTRA_ARGS 变量指定的 IP。
```
[root@master ~]# kubectl get node -o wide
NAME     STATUS   ROLES    AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION              CONTAINER-RUNTIME
master   Ready    master   24m   v1.15.1   192.168.26.10   <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
node1    Ready    <none>   21m   v1.15.1   10.0.2.15       <none>        CentOS Linux 7 (Core)   3.10.0-862.2.3.el7.x86_64   docker://19.3.1
```
用同样的方法修改其他节点 IP 即可。
为了方便，这里提供了一个命令，自动化上面步骤：
```bash
echo KUBELET_EXTRA_ARGS=\"--node-ip=`ip addr show eth1 | grep inet | grep -E -o "([0-9]{1,3}[\.]){3}[0-9]{1,3}/" | tr -d '/'`\" > /etc/sysconfig/kubelet
systemctl restart kubelet
```
