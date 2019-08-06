---
title: Kubeadm 结合 Vagrant 自动化部署最新版 Kubernetes 集群
date: 2019-08-06 14:36:25
categories: Kubernetes
tags: Kubernetes
---

之前写过一篇搭建 k8s 集群的教程：[「使用 kubeadm 搭建 kubernetes 集群」](https://qhh.me/2019/03/19/%E4%BD%BF%E7%94%A8-kubeadm-%E6%90%AD%E5%BB%BA-kubernetes-%E9%9B%86%E7%BE%A4/)，教程中用到了 kubeadm 和 vagrant，但是整个过程还是手动一步一步完成：`创建节点--> 节点配置、相关软件安装  --> 初始化 master 节点 --> node 节点加入 master 节点`。其实这个过程完全可以通过 Vagrant 的配置器自动化来实现，达到的目的是启动一个 k8s 只需在 Vagrant 工程目录执行：vagrant up 即可一键完成集群的创建。

本文主要介绍如何使用 Kubeadm 结合 Vagrant 自动化 k8s 集群的创建，在了解了 kubeadm 手动搭建 kubernetes 集群的过程后，自动化就简单了，如果不了解请参见之前的文章：[「使用 kubeadm 搭建 kubernetes 集群」](https://qhh.me/2019/03/19/%E4%BD%BF%E7%94%A8-kubeadm-%E6%90%AD%E5%BB%BA-kubernetes-%E9%9B%86%E7%BE%A4/)，梳理下整个过程，在此不做过多介绍。下面大概介绍下自动化流程：

1. 首先抽象出来每个节点需要执行的通用脚本，完成一些常用软件的安装、docker 安装、kubectl 和 kubeadm 安装，还有一些节点的系统级配置：
具体实现脚本见：https://github.com/qhh0205/kubeadm-vagrant/blob/master/install-centos.sh

2. 编写 Vagrantfile，完成主节点的初始化安装和 node 节点加入主节点。但是有个地方和之前手动安装不太一样，为了自动化，我们必须在 kubeadm 初始化 master 节点之前生成 TOKEN（使用其他任意主机的 kubeadm 工具生成 TOKEN 即可），然后自动化脚本统一用这个 TOKEN 初始化主节点和从节点加入。Vagrantfile 具体实现见：https://github.com/qhh0205/kubeadm-vagrant/blob/master/Vagrantfile

完整的 Vagrant 工程在这里：https://github.com/qhh0205/kubeadm-vagrant
使用 kubeadm + vagrant 自动化部署 k8s 集群，基于 Centos7 操作系统。该工程 fork 自 [kubeadm-vagrant](https://github.com/coolsvap/kubeadm-vagrant), 对已知问题进行了修复：节点设置正确的 IP 地址[「set-k8s-node-ip.sh」](https://github.com/qhh0205/kubeadm-vagrant/blob/master/set-k8s-node-ip.sh)。否则使用过程中会出现问题，具体问题见这里：[「kubeadm + vagrant 部署多节点 k8s 的一个坑」](https://qhh.me/2019/08/06/kubeadm-vagrant-%E9%83%A8%E7%BD%B2%E5%A4%9A%E8%8A%82%E7%82%B9-k8s-%E7%9A%84%E4%B8%80%E4%B8%AA%E5%9D%91/)。其他一些调整：节点初始化脚本更改、Vagrantfile 添加 Shell 脚本配置器，运行初始化脚本。

>默认：1 个 master 节点，1 个 node 节点，可以根据需要修改 Vagrantfile 文件，具体见工程 [README.md](https://github.com/qhh0205/kubeadm-vagrant/blob/master/README.md) 说明。
