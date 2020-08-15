---
title: 使用 kind 1 分钟启动一个本地 k8s 开发集群
date: 2020-08-15 12:40:50
categories: Kubernetes
tags: Kubernetes
---

####  kind 简介
Github 地址：https://github.com/kubernetes-sigs/kind
[kind](https://github.com/kubernetes-sigs/kind) 是一个快速启动 kubernetes 集群的工具，适合本地 k8s 开发环境搭建，能在 1 分钟之内就启动一个非常轻量级的 k8s 集群。之所以如此之快，得益于其基于其把整个 k8s 集群用到的组件都封装在了 Docker 容器里，构建一个 k8s 集群就是启动一个 Docker 容器，如此简单，正如下面图片描述一样：

![Alt text](/images/kind-k8s.png)

说说我为什么使用 kind 吧：
我之前本地 k8s 开发环境是基于 vagrant + virtualbox 来搭建，但是和 kind 比起来太重量级了，主要有如下痛点：
- 资源消耗严重：vagrant + virtualbox + kubernetes 这一套其实本质上还是 k8s 运行在 virtualbox 虚拟机上，这个资源消耗可想而知，我的电脑配置低，经常由于资源消耗太多导致电脑发热、风扇狂转、死机。。。
- 使用复杂：vagrant + virtualbox + kubernetes 虽然比直接二进制搭建简化了不少，但是还是有一定的技术门槛的，需要对 vagrant 有一定的了解，而且编写 vagrant 需要有一定的经验才能把集群配置好；

#### kind 使用
kind 的使用非常简单，其实就是一个命令行工具，通过这个工具创建、删除 k8s 集群，下面简单说下使用。

1.准备工作
- Kind 的主要功能目前需要有 Docker 环境的支持，可参考 [Docker 官方文档](https://docs.docker.com/get-docker/)进行安装；
- 安装操作 k8s 的 kubectl 命令行，安装方法可参考[官方文档](https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl)；

2.安装
到 github release 页下载对应操作系统的二进制文件到本地，并放到 PATH 环境变量：
https://github.com/kubernetes-sigs/kind/releases

3.创建集群
```bash
kind create cluster
```
完成后就可以直接 kubectl 操作集群了，kubeconfig 已经自动生效了，在 `~/.kube/config` 路径。
```bash
$ kubectl get node
NAME                 STATUS    ROLES     AGE       VERSION
kind-control-plane   Ready     master    53m       v1.18.2
```
4.获取集群 kubeconfig 文件
```bash
kind get kubeconfig
```

5.销毁集群
```
kind delete cluster
```
执行 `kind --help` 获取帮助和其他支持的命令参数。

#### kind 配置
我们可以对要创建的集群进行一些定制化配置，kind 支持的配置见这里：https://kind.sigs.k8s.io/docs/user/configuration/
配置方法：
参照文档，编写配置 kind 配置文件 config.yaml，然后在 kind 创建集群的时候指定配置文件：
```bash
kind create cluster --config=/foo/bar/config.yaml
```
举例：
默认 kind 创建出来的集群 apiserver 监听的地址是：127.0.0.1:[随机端口]，我要改成默认监听的地址是：0.0.0.0:6443，编写如下 config.yaml
```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
networking:
  # WARNING: It is _strongly_ recommended that you keep this the default
  # (127.0.0.1) for security reasons. However it is possible to change this.
  apiServerAddress: "0.0.0.0"
  # By default the API server listens on a random open port.
  # You may choose a specific port but probably don't need to in most cases.
  # Using a random port makes it easier to spin up multiple clusters.
  apiServerPort: 6443
```
创建集群时指定上面配置文件：
```bash
kind create cluster --config=config.yaml
```

#### 总结
总而言之，kind 是一个非常轻量级的工具，能以最轻量的方式构建 k8s 集群，其设计理念是是一个 k8s 就是一个 Docker 容器，构建复杂的 k8s 集群变成了启动一个 Docker 容器，如此简单。

我对 kind 的评价是：快！快！快！ 轻量！轻量！轻量！ 效率！效率！效率！
