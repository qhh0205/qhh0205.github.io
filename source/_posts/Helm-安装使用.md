---
title: Helm 安装使用
date: 2019-08-08 10:52:59
categories: Kubernetes
tags: Kubernetes
---

其实 Helm 的安装很简单，之所以单独写这篇文章主要是因为国内网络原因导致 helm 使用存在障碍(防火墙对 google 不友好)，本文重点说如何解决这一问题。
#### helm 安装
官方提供了一件安装脚本，安装最新版：https://helm.sh/docs/using_helm/#installing-helm
```
curl -L https://git.io/get_helm.sh | bash
```

#### 创建服务账号和角色绑定
rbac-config.yaml:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```
```bash
kubectl create -f rbac-config.yaml
```

#### helm init 初始化
国内无法访问 gcr.io 仓库，指定阿里云镜像仓库，同时指定前面创建的服务账号：
```
helm init --service-account tiller -i registry.aliyuncs.com/google_containers/tiller:v2.14.3
```
国内也无法访问 helm 默认的 Chart 仓库，所以也改成阿里云 Chart 镜像仓库：
```bash
helm repo remove stable
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```
