---
title: helm 部署 kubernetes-dashboard
date: 2019-08-08 12:56:56
categories: Kubernetes
tags: Kubernetes
---

[kubernetes-dashboard](https://github.com/kubernetes/dashboard) 是 k8s 官方提供的集群 Web UI，可以查看集群详细的信息，比如集群的 api 资源，pod 日志，工作负载，节点资源利用率等等。在部署 kubernetes-dashboard 前需要先安装 [heapster](https://github.com/kubernetes-retired/heapster) ，heapster 用于收集数据，而 dashboard 是展示数据的界面。关于 heapster 的安装见之前文章[「helm 部署 heapster 组件」](https://qhh.me/2019/08/08/helm-%E9%83%A8%E7%BD%B2-heapster-%E7%BB%84%E4%BB%B6/)。如果没有部署 heapster 组件，kubernetes-dashboard 的 pod 会报错：
>2019/08/06 10:30:06 Metric client health check failed: the server could not find the requested resource (get services heapster). Retrying in 30 seconds.

![](/images/kube-dashboard.png)
使用官方提供的 Chart：https://github.com/helm/charts/tree/master/stable/kubernetes-dashboard
对 values 文件进行一些定制：
- docker 镜像地址改为阿里云镜像地址，国内访问不了默认的镜像地址；
- service 类型改为 NodePort，并指定 nodePort 端口为 30000；

定制后的 values 文件：https://raw.githubusercontent.com/qhh0205/helm-charts/master/kube-component-values/kube-dashboard.yml

helm 部署:
```bash
helm install stable/kubernetes-dashboard  --name kubernetes-dashboard -f  https://raw.githubusercontent.com/qhh0205/helm-charts/master/kube-component-values/kube-dashboard.yml --namespace kube-system
```

创建集群服务账号：admin
```bash
kubectl create -f https://raw.githubusercontent.com/qhh0205/helm-charts/master/some-apiserver-rs/admin-sa.yml
```

获取 dashboard 访问 token：
```bash
kubectl get secret `kubectl get secret -n kube-system | grep admin-token | awk '{print $1}'` -o jsonpath={.data.token} -n kube-system | base64 -d
```
