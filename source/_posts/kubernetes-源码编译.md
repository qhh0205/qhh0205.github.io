---
title: kubernetes 源码编译
date: 2020-03-01 18:15:52
categories: Kubernetes
tags: Kubernetes
---

kubernetes 源码编译分为本地编译和镜像编译，本地编译是指最终编译出来的是二进制可执行文件，镜像编译是
最终编译出来的产出物为 docker 镜像 tar 包。本文主要介绍本地编译的方法，以编译 kube-apiserver 组件为例说明。

### 环境要求
- Go 环境: go1.12.xx
- gcc

> 我的环境说明：Mac Os + go1.12.10 + gcc，如果读者本地的 Go 版本不是 go1.12.xx，可以使用 gvm 工具安装一个，gvm 是 Go 多版本管理工具，具体使用方法可以看之前的文章：[Golang 多版本管理神器 gvm](https://qhh0205.github.io/2020/03/01/Golang-%E5%A4%9A%E7%89%88%E6%9C%AC%E7%AE%A1%E7%90%86%E7%A5%9E%E5%99%A8-gvm/)

本地编译也分为两种，一种是 make 编译，另一种是 Go 命令行编译，下面一一介绍：
#### 一、make 编译
1.下载 k8s 源码
```bash
go get k8s.io/kubernetes
```
2.编译指定版本源码(以1.16.3为例)
```bash
cd $GOPATH/src/k8s.io/kubernetes
git checkout tags/v1.16.3
```
3.设置要编译的组件(以编译 kube-apiserver 组件为例说明)
> mac 下编译要安装 GNU tar: sudo brew install gnu-tar

```bash
make clean && make WHAT=cmd/kube-apiserver
```
编译产出物会在 _output/bin 目录生成:
```bash
$ ls -1 _output/bin
conversion-gen
deepcopy-gen
defaulter-gen
go-bindata
go2make
kube-apiserver
openapi-gen
```

#### 二、Go 命令编译
1.cd $GOPATH/src/k8s.io/kubernetes && make generated_files
2.进入对应组件目录编译(以 kube-apiserver组件编译为例)
```bash
cd cmd/kube-apiserver && go build -v
```

### 源码编译可能遇到的问题
编译可能报类似下面错误：
```bash
go/build: importGo k8s.io/kubernetes: exit status 1
can't load package: package k8s.io/kubernetes: cannot find module providing package k8s.io/kubernetes

+++ [0301 18:04:57] Building go targets for darwin/amd64:
    ./vendor/k8s.io/code-generator/cmd/deepcopy-gen
can't load package: package k8s.io/kubernetes/vendor/k8s.io/code-generator/cmd/deepcopy-gen: cannot find module providing package k8s.io/kubernetes/vendor/k8s.io/code-generator/cmd/deepcopy-gen
!!! [0301 18:05:04] Call tree:
!!! [0301 18:05:04]  1: 
```
解决办法：golang 版本换成 go1.12.xx 即可。
具体 issue 见这里：[https://github.com/kubernetes/kubernetes/issues/84224](https://github.com/kubernetes/kubernetes/issues/84224)

### 参考资料
[https://www.kubernetes.org.cn/5033.html](https://www.kubernetes.org.cn/5033.html)
[https://blog.csdn.net/boling_cavalry/article/details/88591982](https://blog.csdn.net/boling_cavalry/article/details/88591982)
[https://github.com/MicrosoftDocs/Virtualization-Documentation/blob/master/virtualization/windowscontainers/kubernetes/compiling-kubernetes-binaries.md](https://github.com/MicrosoftDocs/Virtualization-Documentation/blob/master/virtualization/windowscontainers/kubernetes/compiling-kubernetes-binaries.md)

