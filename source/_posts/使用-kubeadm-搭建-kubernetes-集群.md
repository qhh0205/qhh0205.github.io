---
title: 使用 kubeadm 搭建 kubernetes 集群
date: 2019-03-19 22:50:45
categories: Kubenetes
tags: Kubenetes
---

### kubeadm 简介
[kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/) 是 [k8s 官方](https://kubernetes.io/)提供的用于快速部署 k8s 集群的命令行工具，也是官方推荐的最小化部署 k8s 集群的最佳实践，比起直接用二进制部署能省去很多工作，因为该方式部署的集群的各个组件以 docker 容器的方式启动，而各个容器的启动都是通过该工具配自动化启动起来的。

kubeadm 不仅仅能部署 k8s 集群，还能很方便的管理集群，比如集群的升级、降级、集群初始化配置等管理操作。

kubeadm 的设计初衷是为新用户提供一种便捷的方式来首次试用 Kubernetes， 同时也方便老用户搭建集群测试他们的应用。

kubeadm 的使用案例：
- 新用户可以从 kubeadm 开始来试用 Kubernetes。
- 熟悉 Kubernetes 的用户可以使用 kubeadm 快速搭建集群并测试他们的应用。
- 大型的项目可以将 kubeadm 和其他的安装工具一起形成一个比较复杂的系统。

### 安装环境要求
- 一台或多台运行着下列系统的机器:
  - Ubuntu 16.04+
  - Debian 9
  - CentOS 7
  - RHEL 7
  - Fedora 25/26
  - HypriotOS v1.0.1+
  - Container Linux (针对1800.6.0 版本测试)
- 每台机器 2 GB 或更多的 RAM (如果少于这个数字将会影响您应用的运行内存)
- 2 CPU 核心或更多(节点少于 2 核的话新版本 kubeadm 会报错)
- 集群中的所有机器的网络彼此均能相互连接(公网和内网都可以)
- 禁用 Swap 交换分区。（Swap 分区必须要禁掉，否则安装会报错）

### 准备环境
本文使用 kubeadm 部署一个 3 节点的 k8s 集群：1 个 master 节点，2 个 node 节点。各节点详细信息如下：

| Hostname | IP | OS 发行版 | 内存（GB）| CPU（核）|
| :----- | :----- | :----- | :----- | :----- |
| k8s-master | 192.168.10.100 | Centos7 | 2 | 2 |
| k8s-node-1 | 192.168.10.101 | Centos7 | 2 | 2 |
| k8s-node-2 | 192.168.10.102 | Centos7 | 2 | 2 |
![](/images/k8s-arch.png)
### kubeadm 安装 k8s 集群完整流程
- 使用 Vagrant 启动 3 台符合上述要求的虚拟机
- 调整每台虚拟机的服务器参数
- 各节点安装 docker、kubeadm、kubelet、kubectl 工具
- 使用 kubeadm 部署 master 节点
- 安装 Pod 网络插件（CNI）
- 使用 kubeadm 部署 node 节点

接下来我们依次介绍每步的具体细节：

#### 使用 Vagrant 启动 3 台符合上述要求的虚拟机
Vagrant 的使用在这里不具体介绍了，如需了解请点击[这里](https://blog.csdn.net/qianghaohao/article/details/80038096)。
本文用到的 Vagrantfile:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
# author: qhh0205

$num_nodes = 2

Vagrant.configure("2") do |config|
  # k8s 主节点定义及初始化配置
  config.vm.define "k8s-master" do | k8s_master |
    k8s_master.vm.box = "Centos7"
    k8s_master.vm.hostname = "k8s-master"
    k8s_master.vm.network "private_network", ip: "192.168.10.100"
    k8s_master.vm.provider "virtualbox" do | v |
      v.name = "k8s-master"
      v.memory = "2048"
      v.cpus = 2
    end
  end

  # k8s node 节点定义及初始化配置
  (1..$num_nodes).each do |i|
      config.vm.define "k8s-node-#{i}" do |node|
        node.vm.box = "Centos7"
        node.vm.hostname = "k8s-node-#{i}"
        node.vm.network "private_network", ip: "192.168.10.#{i+100}"
        node.vm.provider "virtualbox" do |v|
          v.name = "k8s-node-#{i}"
          v.memory = "2048"
          v.cpus = 2
        end
      end
  end
end
```

进入 Vagrantfile 文件所在目录，执行如下命令启动上述定义的 3 台虚拟机：
```bash
vagrant up
```

#### 调整每台虚拟机的服务器参数
1. 禁用 swap 分区：
临时禁用：`swapoff -a`
永久禁用：`sed -i '/swap/s/^/#/g' /etc/fstab`
swap 分区必须禁止掉，否则 `kubadm init` 自检时会报如下错误：
```bash
[ERROR Swap]: running with swap on is not supported. Please disable swap
```
2. 将桥接的 IPv4 流量传递到 iptables 的链：
```bash
$ cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sysctl --system
```
	如果不进行这一步的设置，`kubadm init` 自检时会报如下错误：
```bash
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1
```
3. 关闭网络防火墙：
```
systemctl stop firewalld
systemctl disable firewalld
```
4. 禁用 SELinux：
临时关闭 selinux（不需要重启主机）: `setenforce 0`
永久关闭 selinux（需要重启主机才能生效）：`sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config`

#### 各节点安装 docker、kubeadm、kubelet、kubectl 工具
##### 安装 Docker
配置 Docker yum 源（阿里 yum 源）：
```bash
curl -sS -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
安装 docker：
```bash
yum install --nogpgcheck -y yum-utils device-mapper-persistent-data lvm2
yum install --nogpgcheck -y docker-ce
systemctl enable docker && systemctl start docker
```
##### 安装 kubeadm、kubelet、kubectl 工具
配置相关工具 yum 源（阿里 yum 源）：
```bash
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```
安装 kubeadm、kubelet、kubectl：
其实 kubeadm、kubelet、kubectl 这三个工具的版本命名是一致的（和 k8s 版本命名一致），我们可以指定安装特定的版本，即安装指定版本的 k8s 集群。

查看哪些版本可以安装：
`yum --showduplicates list kubeadm|kubelet|kubectl`

在这里我们安装 `1.13.2` 版本：
```bash
yum install -y kubeadm-1.13.2 kubelet-1.13.2 kubectl-1.13.2
# 设置 kubelet 开机自启动: kubelet 特别重要，如果服务器重启后 kubelet
# 没有启动，那么 k8s 相关组件的容器就无法启动。在这里不需要把 kubelet 启动
# 起来，因为现在还启动不起来，后续执行的 kubeadm 命令会自动把 kubelet 拉起来。
systemctl enable kubelet
```

#### 使用 kubeadm 部署 master 节点
登陆 master 节点执行如下命令：
```bash
kubeadm init --kubernetes-version v1.13.2 --image-repository registry.aliyuncs.com/google_containers --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.168.10.100
```
参数说明：
`--kubernetes-version`:  安装指定版本的 k8s 版本，该参数和 kubeadm 的版本有关，特定版本的 kubeadm 并不能安装所有版本的 k8s，最好还是 kubeadm 的版本和该参数指定的版本一致。

`--image-repository`: 该参数仅在高版本（具体哪个版本没仔细查，反正在 1.13.x 中是支持的）的 kubeadm 中支持，用来设置 kubeadm 拉取 k8s 各组件镜像的地址，默认拉取的地址是：[k8s.gcr.io](k8s.gcr.io)。众所周知 [k8s.gcr.io](k8s.gcr.io) 国内是无法访问的，所以在这里改为阿里云镜像仓库。

`--pod-network-cidr`: 设置 pod ip 的网段 ，网段之所以是 `10.244.0.0/16`，是因为后面安装 flannel 网络插件时，yaml 文件里面的 ip 段也是这个，两个保持一致，不然可能会使得 Node 间 Cluster IP 不通。这个参数必须得指定，如果这里不设置的话后面安装 flannel 网络插件时会报如下错误：
```
E0317 17:02:15.077598       1 main.go:289] Error registering network: failed to acquire lease: node "k8s-master" pod cidr not assigned
```
`--apiserver-advertise-address`: API server 用来告知集群中其它成员的地址，这个参数也必须得设置，否则 api-server 容器启动不起来，该参数的值为 master 节点所在的本地 ip 地址。

----
题外话：像之前没有 `--image-repository` 这个参数时，大家为了通过 kubeadm 安装 k8s 都是采用"曲线救国"的方式：先从别的地方把同样的镜像拉到本地（当然镜像的 tag 肯定不是 k8s.gcr.io/xxxx），然后将拉下来的镜像重新打个 tag，tag 命名成和执行 `kubeadm init` 时真正拉取镜像的名称一致（比如：k8s.gcr.io/kube-controller-manager-amd64:v1.13.2）。这么做显然做了很多不必要的工作，幸好现在有了 `--image-repository` 这个参数能自定义 kubeadm 拉取 k8s 相关组件的镜像地址了。

----
执行上面命令后，如果出现如下输出（截取了部分），则表示 master 节点安装成功了：
```bash
...
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 192.168.10.100:6443 --token jm6o42.9ystvjarc6u09pjp --discovery-token-ca-cert-hash sha256:64405f3a90597e0ebf1f33134649196047ce74df575cb1a7b38c4ed1e2f94421
```

根据上面输出知道：要开始使用集群普通用户执行下面命令：
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
现在就可以使用 kubectl 访问集群了：
```bash
[vagrant@k8s-master ~]$ kubectl get node
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   NotReady   master   13m   v1.13.2
```
可以看出现在 master 节点还是 `NotReady` 状态，这是因为默认情况下，为了保证 master 的安全，master 是不会被分配工作负载的。你可以取消这个限制通过输入（不建议这样做，我们后面会向集群中添加两 node 工作节点）：
```bash
$ kubectl taint nodes --all node-role.kubernetes.io/master-
```

#### 安装 Pod 网络插件（CNI）
Pod 网络插件有很多种，具体见这里：https://kubernetes.io/docs/concepts/cluster-administration/addons/，我们选择部署 [Flannel](https://github.com/coreos/flannel) 网络插件：
```bash
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

#### 使用 kubeadm 部署 node 节点
前面已经将 master 节点部署完了，接下来部署 node 节点就很简单了，在 node节点执行如下命令将自己加到 k8s 集群中（复制 master 节点安装完后的输出）：
```bash
kubeadm join 192.168.10.100:6443 --token jm6o42.9ystvjarc6u09pjp --discovery-token-ca-cert-hash sha256:64405f3a90597e0ebf1f33134649196047ce74df575cb1a7b38c4ed1e2f94421
```
出现如下输出（截取了部分）表示成功将 node 添加到了集群：
```bash
...
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```
在 master 节点查看 node 状态，如果都为 Ready，则表示集群搭建完成：
```bash
[vagrant@k8s-master ~]$ kubectl get nodes
NAME         STATUS   ROLES    AGE   VERSION
k8s-master   Ready    master   14m   v1.13.2
k8s-node-1   Ready    <none>   3m    v1.13.2
```
用同样的方法把另一个节点也加入到集群中。

### 相关资料
https://purewhite.io/2017/12/17/use-kubeadm-setup-k8s/
https://kubernetes.io/zh/docs/setup/independent/install-kubeadm/
https://k8smeetup.github.io/docs/admin/kubeadm/
