---
title: 基于 Kubernetes 的动态 Jenkins slave 部署
date: 2018-10-14 10:10:36
categories: DevOps
tags: Jenkins
---

采用官方 Helm Chart 部署，服务对外暴露方式为 KongIngress.

官方 Jenkins Chart 仓库：https://github.com/helm/charts/tree/master/stable/jenkins

### 1. 创建 jenkins pv
pv 底层类型为 nfs
jenkins_pv.yaml:
```yaml
kubectl create -f jenkins_pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  labels:
    app: jenkins
spec:
  capacity:
    storage: 50Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /data1/nfs/jenkins
    server: 10.4.37.91
```
### 2. 创建 namespace
```
kubectl create ns jenkins
```
### 3. 采用 KongIngress 方式对外暴露服务
修改 values.yml 文件:
3.1. Master.ServiceType 改为 ClusterIP
3.2. HostName 取消注释，值设置为 Jenkins 访问域名：example.com
3.3. rbac 设置为 true；
3.4. Master.Ingress.Annotations 添加如下内容：
```yaml
ingress.plugin.konghq.com: jenkins-kong-ingress
kubernetes.io/ingress.class: nginx
```
3.5. values.yaml Master 节点下添加 Kong Ingress 相关变量
```yaml
    KongIngress:
      Name: jenkins-kong-ingress
      Route:
        StripPath: true
        PreserveHost: true
      Proxy:
        ConnectTimeout: 10000
        Retries: 5
        ReadTimeout: 60000
        WriteTimeout: 60000
```
### 4. 编辑 jenkins-master-ingress.yaml 添加  KongIngress 资源对象
```yaml
apiVersion: configuration.konghq.com/v1
kind: KongIngress
metadata:
  name: {{ .Values.Master.KongIngress.Name }}
route:
  strip_path: {{ .Values.Master.KongIngress.Route.StripPath }}
  preserve_host: {{ .Values.Master.KongIngress.Route.PreserveHost }}
proxy:
  connect_timeout: {{ .Values.Master.KongIngress.Proxy.ConnectTimeout }}
  retries: {{ .Values.Master.KongIngress.Proxy.Retries }}
  read_timeout: {{ .Values.Master.KongIngress.Proxy.ReadTimeout }}
  write_timeout: {{ .Values.Master.KongIngress.Proxy.WriteTimeout }}
```
### 5. helm 打包
```
helm package jenkins
```

### 6. 重新生成 chart 索引
```
helm repo index .
```

### 7. helm 部署
```
helm install --name jenkins helm_local_repo/jenkins --namespace jenkins
```

### 8. 获取 Jenkins 初始密码
 执行 `kubectl get secret jenkins -n jenkins -o yaml` 得到 jenkins-admin-password 的 base64 编码值，然后通过 base64 解码，得到密码：
```
echo 'base64d_str' | base64 -d
```
## 参考链接
https://mp.weixin.qq.com/s/OoTEtPNEORn_sFYG8rzaqA

