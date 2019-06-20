---
title: 非容器化 gitlab 进行容器化改造
date: 2018-12-02 16:40:42
categories: Git
tags: Git
---

本文主要介绍非容器化（通过 yum 在 Linux 服务器安装）gitlab 进行容器化改造的两种方法，都是基于 Kubernetes 平台，均采用 helm 部署。第一种是基于自建 k8s 平台部署 gitlab，第二种是基于 Google GKE 平台部署 gitlab。

Docker 镜像采用[基于 Omnibus 安装包的镜像](https://hub.docker.com/r/gitlab/gitlab-ee)，gitlab 的各个组件都运行在同一个容器中。关于 GitLab Ominibus 镜像和云原生镜像的区别见[这里](https://docs.gitlab.com/ee/install/docker.html)。

### gitlab 容器化改造（基于自建 k8s 平台部署 gitlab）
#### 一、搭建和原先版本一致的 gitlab
github helm gitlab-ee chart：https://github.com/helm/charts/tree/master/stable/gitlab-ee

在此 helm chart 基础上将备份目录也(/var/opt/gitlab/backups)通过PVC持久化，方便数据的备份恢复: https://github.com/qhh0205/helm-charts/tree/master/gitlab-ee
1. 手动创建需要的 pv（基于 nfs）
https://github.com/qhh0205/kubernetes-resources/tree/master/gitlab-pv
2. 部署
其他自定义参数修改 values-custom.yaml 文件，比如镜像版本、硬件配置等参数。
```bash
git clone git@github.com:qhh0205/helm-charts.git
cd helm-charts/gitlab-ee
helm install --name gitlab --set externalUrl=http://domain/,gitlabRootPassword=xxxx -f values-custom.yaml ./ --namespace=gitlab
```

#### 二、数据恢复
1. 拷贝 gitlab 备份文件到容器外挂nfs目录(/data/nfs/gitlab/gitlab-data-backups（nfs路径）--->/var/opt/gitlab/backups（容器路径）)；
2. 进入容器：
kubectl exec -it pod_name /bin/sh -n gitlab 
3. gitlab-ctl reconfigure
4. chown git:git 1543237967_2018_11_26_10.1.3-ee_gitlab_backup.tar
5. chown -R git:root /gitlab-data # 由于 gitlab-rake 执行过程中 默认用户名是 git，所以需要把该目录的属主改成 git，否则恢复时报错权限问题；
6. gitlab-rake gitlab:backup:restore
根据提示输入相关信息
YES
YES
gitlab-ctl restart 

### gitlab 容器化改造（基于 Google 云 GKE 平台）
#### 一、搭建和原先版本一致的 gitlab
github helm gitlab-ee chart：https://github.com/helm/charts/tree/master/stable/gitlab-ee

在此 helm chart 基础上将备份目录也(/var/opt/gitlab/backups)通过PVC持久化，方便数据的备份恢复: 
https://github.com/qhh0205/helm-charts/tree/master/gitlab-ee

其他自定义参数修改 values-custom.yaml 文件，比如镜像版本、硬件配置等参数。
```bash
git clone git@github.com:qhh0205/helm-charts.git
cd helm-charts/gitlab-ee
helm install --name gitlab --set externalUrl=http://domain/,gitlabRootPassword=xxxx -f values-custom.yaml ./ --namespace=gitlab
```

#### 二、数据迁移恢复
1. 将 gitlab 备份文件拷贝到 k8s gitlab pod 容器目录：
```bash
kubectl cp 1543237967_2018_11_26_10.1.3-ee_gitlab_backup.tar \
namespace/pod_name:/var/opt/gitlab/backups -n gitlab
```
2. gitlab-ctl reconfigure
3. chown git:git 1543237967_2018_11_26_10.1.3-ee_gitlab_backup.tar
4. chown -R git:root /gitlab-data # 由于 git-rake 执行过程中 默认用户名是 git，所以需要把该目录的属主改成git，否则恢复时报错权限问题；
5. gitlab-rake gitlab:backup:restore
根据提示输入相关信息
YES
YES
gitlab-ctl restart 

### 外部访问
1. Kong Ingress
2. NodePort
3. LoadBalancer（云提供商平台，比如 Google GKE）

### 相关链接
- 容器化安装 gitlab：https://docs.gitlab.com/ee/install/docker.html
- gitlab 数据存放目录修改：https://blog.whsir.com/post-1490.html
- gitlab 安装软件和硬件需求：https://docs.gitlab.com/ce/install/requirements.html
- Omnibus GitLab documentation: https://docs.gitlab.com/omnibus/README.html
