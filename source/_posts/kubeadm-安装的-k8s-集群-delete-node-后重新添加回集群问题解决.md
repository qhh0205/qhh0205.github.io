---
title: kubeadm 安装的 k8s 集群 delete node 后重新添加回集群问题解决
date: 2018-09-11 14:08:54
categories: Kubernetes
tags: k8s
---

#### 问题描述
前不久公司同事误操作，直接 kubectl delete node node_ip 从集群中删除了一个 node，后来未知原因服务器给宕机了，重启服务器后 docker、kubelet 等服务器都自动重启了（用 systemd 管理），但是 node 一直是 Not Ready 状态，按理来说执行如下命令把节点添加回集群即可：
```
kubeadm join --token xxxxxxx master_ip:6443 --discovery-token-ca-cert-hash sha256:xxxxxxx
```
但是执行如上命令后报错如下(提示 10250 端口被占用)：
```
[root@com10-81 ~]# kubeadm join --token xxxx 10.4.37.167:6443 --discovery-token-ca-cert-hash sha256:xxxxxx
[preflight] Running pre-flight checks.
	[WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.12.1-ce. Max validated version: 17.03
	[WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Some fatal errors occurred:
	[ERROR Port-10250]: Port 10250 is in use
	[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
	[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
```
#### 解决方法
出现如上问题的主要原因是之前 kubeadm init 初始化过，所以一些配置文件及服务均已存在，重新执行 kubeadm join 时必然
会导致冲突，解决方法如下：
1.先执行 kubeadm reset，重新初始化节点配置：
`kubeadm reset`
```
[root@com10-81 ~]# kubeadm reset
[preflight] Running pre-flight checks.
[reset] Stopping the kubelet service.
[reset] Unmounting mounted directories in "/var/lib/kubelet"
[reset] Removing kubernetes-managed containers.
[reset] No etcd manifest found in "/etc/kubernetes/manifests/etcd.yaml". Assuming external etcd.
[reset] Deleting contents of stateful directories: [/var/lib/kubelet /etc/cni/net.d /var/lib/dockershim /var/run/kubernetes]
```
2.然后执行 kubeadm join 添加节点到集群（如果 token 失效，到主节点执行：kubeadm token create 重新生成）：
`kubeadm join --token xxxxx master_ip:6443 --discovery-token-ca-cert-hash sha256:xxxx`
```
[root@com10-81 ~]# kubeadm join --token xxxxx 10.4.37.167:6443 --discovery-token-ca-cert-hash sha256:xxxxxxx
[preflight] Running pre-flight checks.
	[WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 17.12.1-ce. Max validated version: 17.03
	[WARNING FileExisting-crictl]: crictl not found in system path
[preflight] Starting the kubelet service
[discovery] Trying to connect to API Server "10.4.37.167:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.4.37.167:6443"
[discovery] Requesting info from "https://10.4.37.167:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "10.4.37.167:6443"
[discovery] Successfully established connection with API Server "10.4.37.167:6443"

This node has joined the cluster:
* Certificate signing request was sent to master and a response
  was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```
PS: k8s 集群 /etc/kubernetes/pki/ca.crt 证书(任何一节点都有该文件) sha256 编码获取（kubeadm join 添加集群节点时需要该证书的 sha256 编码串认证）：
`openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'`

到此节点添加回集群了，但是直接执行 kubectl 相关的命令可能还会报如下错误：
```
[root@com10-81 ~]# kubectl get pod
The connection to the server localhost:8080 was refused - did you specify the right host or port?
You have mail in /var/spool/mail/root
```
问题原因及解决方法:
很明显 kubelet 加载的配置文件(/etc/kubernetes/kubelet.conf)有问题，可能服务器重启的缘故，启动后该文件丢失了，导致里面的连接 master 节点的配置及其他配置给丢了，因此会默认连接 localhost:8080 端口。解决方法很简单：拷贝其他任一节点的该文件，然后重启 kubelet (systemctl restart kublete)即可。

#### 参考链接
https://stackoverflow.com/questions/41732265/how-to-use-kubeadm-to-create-kubernetest-cluster
https://blog.csdn.net/mailjoin/article/details/79686934
