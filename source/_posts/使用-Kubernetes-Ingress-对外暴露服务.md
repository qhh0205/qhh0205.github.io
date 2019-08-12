---
title: 使用 Kubernetes Ingress 对外暴露服务
date: 2019-08-12 22:48:56
categories: Kubernetes
tags: Kubernetes
---

本文主要介绍如何通过 Kubernetes Ingress 资源对象实现从外部对 k8s 集群中服务的访问，介绍了 k8s 对外暴露服务的多种方法、Ingress 及 Ingress Controller 的概念。涉及到的话题有：
- k8s 对外暴露服务的方法；
- Ingress 及 Ingress Controller 简介；
- helm 裸机部署 Nginx Ingress Controller；
- 使用 Ingress 对外暴露服务；
- 通过 Ingress 访问 kubernetes dashboard（支持 HTTPS 访问）；

### k8s 对外暴露服务的方法
向 k8s 集群外部暴露服务的方式有三种： nodePort，LoadBalancer 和本文要介绍的 Ingress。每种方式都有各自的优缺点，nodePort 方式在服务变多的情况下会导致节点要开的端口越来越多，不好管理。而 LoadBalancer 更适合结合云提供商的 LB 来使用，但是在 LB 越来越多的情况下对成本的花费也是不可小觑。Ingress 是 k8s 官方提供的用于对外暴露服务的方式，也是在生产环境用的比较多的方式，一般在云环境下是 LB + Ingress Ctroller 方式对外提供服务，这样就可以在一个 LB 的情况下根据域名路由到对应后端的 Service，有点类似于 Nginx 反向代理，只不过在 k8s 集群中，这个反向代理是集群外部流量的统一入口。

### Ingress 及 Ingress Controller 简介
Ingress 是 k8s 资源对象，用于对外暴露服务，该资源对象定义了不同主机名（域名）及 URL 和对应后端 Service（k8s Service）的绑定，根据不同的路径路由 http 和 https 流量。而 Ingress Contoller 是一个 pod 服务，封装了一个 web 前端负载均衡器，同时在其基础上实现了动态感知 Ingress 并根据 Ingress 的定义动态生成 前端 web 负载均衡器的配置文件，比如 Nginx Ingress Controller 本质上就是一个 Nginx，只不过它能根据 Ingress 资源的定义动态生成 Nginx 的配置文件，然后动态 Reload。个人觉得 Ingress Controller 的重大作用是将前端负载均衡器和 Kubernetes 完美地结合了起来，一方面在云、容器平台下方便配置的管理，另一方面实现了集群统一的流量入口，而不是像 nodePort 那样给集群打多个孔。

![](/images/ingress.png)

所以，总的来说要使用 Ingress，得先部署 Ingress Controller 实体（相当于前端 Nginx），然后再创建 Ingress （相当于 Nginx 配置的 k8s 资源体现），Ingress Controller 部署好后会动态检测 Ingress 的创建情况生成相应配置。Ingress Controller 的实现有很多种：有基于 Nginx 的，也有基于 HAProxy的，还有基于 OpenResty 的 Kong Ingress Controller 等，更多 Controller 见：[https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)，本文使用基于 Nginx 的 Ingress Controller：[ingress-nginx](https://github.com/nginxinc/kubernetes-ingress)。

### helm 裸机部署 Nginx Ingress Controller
基于 Nginx 的 Ingress Controller 有两种，一种是 k8s 社区提供的 [ingress-nginx](https://github.com/nginxinc/kubernetes-ingress)，另一种是 Nginx 社区提供的 [kubernetes-igress](https://kubernetes.io/docs/concepts/services-networking/ingress/)，关于两者的区别见 [这里](https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/nginx-ingress-controllers.md)。

在这里我们部署 k8s 社区提供的 [ingress-nginx](https://github.com/nginxinc/kubernetes-ingress)，Ingress Controller 对外暴露方式采用 hostNetwork，在裸机环境下更多其他暴露方式见：https://kubernetes.github.io/ingress-nginx/deploy/baremetal/

![](/images/hostNetworkIngress.png)

使用 Helm 官方提供的 Chart [stable/nginx-ingress](https://github.com/helm/charts/tree/master/stable/nginx-ingress)，修改 values 文件：
- 使用 DaemonSet 控制器，默认是 Deployment：controller.kind 设为 DaemonSet；
- pod 使用主机网络：controller.hostNetwork 设为 true；
- 在hostNetwork 下 pod 使用集群提供 dns 服务：controller.dnsPolicy 设为 ClusterFirstWithHostNet；
- Service 类型设为 ClusterIP，默认是 LoadBalancer：controller.service.type 设为 ClusterIP；
- 默认后端镜像使用 docker hub 提供的镜像，Google 国内无法访问；

修改后的 values 文件：https://raw.githubusercontent.com/qhh0205/helm-charts/master/nginx-ingress-values.yml
helm 部署
```bash
helm install stable/nginx-ingress --name nginx-ingress -f https://raw.githubusercontent.com/qhh0205/helm-charts/master/nginx-ingress-values.yml
```
验证部署是否成功
```
[root@master ~]# kubectl get pod
NAME                                             READY   STATUS    RESTARTS   AGE
nginx-ingress-controller-mg8df                   1/1     Running   2          2m14s
nginx-ingress-default-backend-577857cd9c-gfsnd   1/1     Running   0          2m14s
```
浏览器访问节点 ip 出现：`default backend - 404` 页面，部署成功。

> 至此 Nginx Ingress Controller 已部署完成，接下来讲解如何通过 Ingress 结合  Ingress Controller 实现集群服务对外访问。

### 使用 Ingress 对外暴露服务
为了快速体验 Ingress，下面部署一个 nginx 服务，然后通过 Ingress 对外暴露 nginx service 进行访问。
首先部署 nginx 服务：
Deployment + Service：nginx.yml
```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
---
kind: Service
apiVersion: v1
metadata:
  name: nginx
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
```
kubectl create -f nginx.yml

接下来创建 Ingress 对外暴露 nginx service 80 端口：
ingress.yml:
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-nginx
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
spec:
  rules:
  - host: nginx.kube.com
    http:
      paths:
      - path: /
        backend:
          serviceName: nginx
          servicePort: 80
```
说明：
- `kubernetes.io/ingress.class: "nginx"`：Nginx Ingress Controller 根据该注解自动发现 Ingress；
- `host: nginx.kube.com`：对外访问的域名；
- `serviceName: nginx`：对外暴露的 Service 名称；
- `servicePort: 80`：nginx service 监听的端口；

>注意：创建的 Ingress 必须要和对外暴露的 Service 在同一命名空间下！

将域名 nginx.kube.com 绑定到 k8s 任意节点 ip 即可访问：http://nginx.kube.com

![](/images/ingress-demo.png)

>上面的示例不支持 https 访问，下面举一个支持 https 的 Ingress 例子：通过 Ingress 访问 kubernetes dashboard 服务。

### 通过 Ingress 访问 kubernetes dashboard（支持 HTTPS 访问）
之前我们使用 helm 以 nodePort 的方式部署了 kubernetes dashboard：[「helm 部署 kubernetes-dashboard」](https://blog.csdn.net/qianghaohao/article/details/98860671)，从集群外部只能通过 `nodeIP:nodePort 端口号` 访问，接下来基于之前部署的 kubernetes-dashboard 配置如何通过 Ingress 访问，并且支持 HTTPS 访问，HTTP 自动跳转到 HTTPS。 ：
首先，练习使用，先用自签名证书来代替吧：
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout kube-dashboard.key -out kube-dashboard.crt -subj "/CN=dashboard.kube.com/O=dashboard.kube.com"
```

使用生成的证书创建 k8s Secret 资源，下一步创建的 Ingress 会引用这个 Secret：
```bash
kubectl create secret tls kube-dasboard-ssl --key kube-dashboard.key --cert kube-dashboard.crt -n kube-system
```

创建 Ingress 资源对象（支持 HTTPS 访问）：
kube-dashboard-ingress.yml
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: ingress-kube-dashboard
  annotations:
    # use the shared ingress-nginx
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
spec:
  tls:
  - hosts:
    - dashboard.kube.com
    secretName: kube-dasboard-ssl
  rules:
  - host: dashboard.kube.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```
kubectl create -f kube-dashboard-ingress.yml -n kube-system
说明：
- `kubernetes.io/ingress.class: "nginx"`：Inginx Ingress Controller 根据该注解自动发现 Ingress；
- `nginx.ingress.kubernetes.io/backend-protocol`: Controller 向后端 Service 转发时使用 HTTPS 协议，这个注解必须添加，否则访问会报错，可以看到 Ingress Controller 报错日志：kubectl logs -f nginx-ingress-controller-mg8df
> 2019/08/12 06:40:00 [error] 557#557: *56049 upstream sent no valid HTTP/1.0 header while reading response header from upstream, client: 192.168.26.10, server: dashboard.kube.com, request: "GET / HTTP/1.1", upstream: "http://10.244.1.8:8443/", host: "dashboard.kube.com"

	报错原因主要是 dashboard 服务后端只支持 https，但是 Ingress Controller 接到客户端的请求时往后端 dashboard 服务转发时使用的是 http 协议，解决办法就是给 创建的 Ingress 设置：`nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"` 注解。解决方法参考自 StackOverflow：https://stackoverflow.com/questions/48324760/ingress-configuration-for-dashboard
- `secretName: kube-dasboard-ssl`：https 证书 Secret；
- `host: dashboard.kube.com`：对外访问的域名；
- `serviceName: kubernetes-dashboard`：集群对外暴露的 Service 名称；
- `servicePort: 443`：service 监听的端口；

>注意：创建的 Ingress 必须要和对外暴露的 Service 在同一命名空间下！

将域名 dashboard.kube.com 绑定到 k8s 任意节点 ip 即可访问：https://dashboard.kube.com

![](/images/kube-dashboard-ingress.png)

### 相关文档
https://kubernetes.io/docs/concepts/services-networking/ingress/ | 官方文档
https://mritd.me/2017/03/04/how-to-use-nginx-ingress/ | Kubernetes Nginx Ingress 教程
https://github.com/nginxinc/kubernetes-ingress | Inginx Ingress Controller：nginx 社区提供
https://github.com/kubernetes/ingress-nginx | Inginx Ingress Controller：k8s 社区提供
https://github.com/nginxinc/kubernetes-ingress/blob/master/docs/nginx-ingress-controllers.md | 两种基于 nginx 的 Ingress Controller 区别
https://kubernetes.github.io/ingress-nginx/deploy/ | Inginx Ingress Controller k8s 社区版安装文档
https://kubernetes.github.io/ingress-nginx/deploy/baremetal/ | 在 裸机环境下 Inginx Ingress Controller 对外暴露方案

