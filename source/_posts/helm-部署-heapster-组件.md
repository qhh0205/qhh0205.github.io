---
title: helm 部署 heapster 组件
date: 2019-08-08 12:30:15
categories: Kubernetes
tags: Kubernetes
---

之前工作用的 k8s 集群（GKE）都是支持 kubectl top node 查看节点资源使用情况的，最近自己本地新搭的集群发现用不了该命令。网上搜索了下发现是由于缺少集群指标收集组件导致，目前常用的集群指标收集组件是 [heapster](https://github.com/kubernetes-retired/heapster) 和 [metrics-server](https://github.com/kubernetes-incubator/metrics-server/)，看官方介绍 heapster 要逐渐被淘汰了，更推荐 metrics-server。但是为了适配后续要安装的 [kubernetes-dashboard](https://github.com/kubernetes/dashboard)，先使用 heapster 组件。

heapster 组件是 Kubernetes 官方支持的容器集群监控组件，主要用于收集集群指标数据并存储，收集的数据可以对接可视化图表展示分析，比如 grafana、kubernetes-dashboard。

除了 `kubectl top node` 依赖于 heapster 收集的指标，kubernetes-dashboard 也需要 heapster，本文使用 helm 来一键部署 heapster 组件。

heapster Chart 使用 helm 官方提供的：https://github.com/helm/charts/tree/master/stable/heapster
对 values 文件做一些定制：
- 使用阿里云镜像仓库，Chart 默认使用 k8s.gcr.io，国内是拉不下来镜像的；
- rbac 服务账号使用 admin，使用默认的 default 账号权限太小，收集指标时报错。
- heapster 启动参数 --source 调整成：`kubernetes:https://kubernetes.default:443?useServiceAccount=true&kubeletHttps=true&kubeletPort=10250&insecure=true`，否则 heapster 会报错：
>E0806 22:11:05.017143       1 manager.go:101] Error in scraping containers from kubelet_summary:192.168.26.10:10255: Get http://192.168.26.10:10255/stats/summary/: dial tcp 192.168.26.10:10255: getsockopt: connection refused

	报错主要原因是 k8s 1.12.0 以后已经取消了 kubelet 10255 端口:
	```
 --read-only-port int32    
 The read-only port for the Kubelet to serve on with no authentication/authorization 
(set to 0 to disable) (default 10255) (DEPRECATED: 
This parameter should be set via the config file specified by the Kubelet's --config flag. 
See https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/ for more information.)
```
	详情见：https://sealyun.com/post/heapster-error/

定制后的 values 文件在这里：https://raw.githubusercontent.com/qhh0205/helm-charts/master/kube-component-values/heapster-values.yml 

接下来使用 helm 安装部署 heapster：
```
helm install stable/heapster --name heapster -f https://raw.githubusercontent.com/qhh0205/helm-charts/master/kube-component-values/heapster-values.yml --namespace kube-system
```

heapster 安装完后就可以正常使用 kubectl top node 了：
```bash
[root@master ~]# kubectl top node
NAME     CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
master   141m         7%     1096Mi          63%
node1    45m          2%     821Mi           47%
```
如果 heapster pod 有问题一般会报下面错误：
```bash
[root@master ~]# kubectl top node
error: metrics not available yet
```
