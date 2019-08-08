---
title: Vagrant 多网卡环境下 flannel 网络插件导致 DNS 无法解析
date: 2019-08-08 16:08:39
categories: Kubernetes
tags: Kubernetes
---

之前写过一篇 k8s 集群自动化部署的文章：[「Kubeadm 结合 Vagrant 自动化部署最新版 Kubernetes 集群」](https://qhh.me/2019/08/06/Kubeadm-%E7%BB%93%E5%90%88-Vagrant-%E8%87%AA%E5%8A%A8%E5%8C%96%E9%83%A8%E7%BD%B2%E6%9C%80%E6%96%B0%E7%89%88-Kubernetes-%E9%9B%86%E7%BE%A4/)，发现集群启动后 DNS 无法解析，公网和集群内部都无法解析，具体问题表现是：进入 pod 执行 ping service 名称或者公网域名都是无法解析 `Unknow host`。

经过网上搜索一番找到了问题并得以解决，主要原因是 Vagrant 在多主机模式下有多个网卡，eth0 网卡用于 nat 转发访问公网，而 eth1 网卡才是主机真正的 IP，在这种情况下直接部署 k8s flannel 插件会导致 CoreDNS 无法工作。解决方法很简单，调整下 flannel 的启动参数，加上 `- --iface=eth1` 网卡参数：
vim https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
`- --kube-subnet-mgr` 后面加一行参数：`- --iface=eth1`
```bash
......
containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1
......
```

参考文档：
https://blog.frognew.com/2019/07/kubeadm-install-kubernetes-1.15.html#2-3-%E5%AE%89%E8%A3%85pod-network | 使用kubeadm安装Kubernetes 1.1
https://github.com/kubernetes/kubeadm/issues/1056 | github issue 讨论
