---
title: kubectl 多集群访问配置
date: 2019-08-09 10:18:50
categories: Kubernetes
tags: Kubernetes
---

配置 `KUBECONFIG` 环境变量，是 kubectl 工具支持的变量，变量内容是冒号分隔的 kubernetes config 认证文件路径。假如我们有两个集群：A 和 B，A 集群的 config 文件为：`$HOME/.kube/config`，B 集群的 config 文件为：`$HOME/.kube/config-local`。要配置 kubectl 随时在两个集群间切换，只需要设置 `KUBECONFIG` 环境变量为：`$HOME/.kube/config:$HOME/.kube/config-local`
```bash
export KUBECONFIG=$HOME/.kube/config:$HOME/.kube/config-local
```
当进行上面配置后，使用 `kubectl config view` 查看 kubectl 配置时，结果为两个文件的合并。
当需要切换集群时，使用 `kubectl config use-context <context 名称>`

参考文档：
https://kubernetes.io/docs/tasks/access-application-cluster/configure-access-multiple-clusters/
