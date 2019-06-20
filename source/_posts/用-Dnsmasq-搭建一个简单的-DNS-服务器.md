---
title: 用 Dnsmasq 搭建一个简单的 DNS 服务器
date: 2019-04-27 22:25:49
categories: DNS
tags: DNS
---

本文主要介绍如何通过 [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) 工具搭建一个简单的 DNS 服务器，搭建完成后就可以马上测试使用了。

### Dnsmasq 简介
[Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) 是一个轻量级的 DNS 缓存、DHCP、TFTP、PXE 服务器。

作为域名解析服务器，dnsmasq 可以通过缓存 DNS 请求来提高对访问过域名的解析速度。

作为 DHCP 服务器，Dnsmasq 可以用于为局域网电脑分配内网 IP 地址和提供路由。DNS 和 DHCP 两个功能可以同时或分别单独实现。

### Dnsmasq 的应用场景
我们一般使用 Dnsmasq 的 DNS 功能，总结了下基于该功能有如下使用场景：
- 作为内部局域网的一个 DNS 缓存服务器。通过 DNS 缓存的功能，可以提高应用程序域名解析的速度。比如 [Kubernetes](https://kubernetes.io/) 的 [kube-dns](https://github.com/kubernetes/dns) 组件中就用 dnsmasq 容器作为 DNS 服务器，用 kube-dns 容器作为 dnsmasq 的上游服务器。dnsmasq 本身具有缓存功能，所以可以大大提高集群中服务名的解析速度，而不需要每次解析请求都访问 kube-dns 容器。

- 实现 DNS 劫持功能。在局域网中，我们有时候可能希望暂时将某个公网域名解析到一个临时的地址，不走公网 DNS。


### Dnsmasq 的工作原理
Dnsmasq 在接受到用户的一个 DNS 请求时，首先会查找 /etc/hosts 这个文件，如果 /etc/hosts 文件没有请求的记录，然后查找 /etc/resolv.conf 中定义的外部 DNS（也叫上游 DNS 服务器，nameserver 配置），外部 DNS 通过递归查询查找到请求后响应给客户端，然后 dnsmasq 将请求结果缓存下来（缓存到内存）供后续的解析请求。

配置 Dnsmasq 为 DNS 缓存服务器，同时在 /etc/hosts 文件中加入本地内网解析，这样一来每当内网机器查询时就会优先查询 hosts 文件，这就等于将 /etc/hosts 共享给全内网机器使用，从而解决内网机器互相识别的问题。相比逐台机器编辑 hosts 文件或者添加 Bind DNS 记录，仅编辑一个 hosts 文件，这简直太容易了。

### Dnsmasq 安装
Dnsmasq 的安装特别简单，以 Centos7 下安装为例：
```bash
sudo yum install -y dnsmasq 
```

### Dnsmasq 配置及启动
#### 配置
Dnsmasq 的所有的配置都在 `/etc/dnsmasq.conf` 这一个文件中完成 。官方在配置文件 `/etc/dnsmasq.conf` 中针对选项和参数等做了比较好的注释说明，我们可以将配置做一次备份，以便以后查阅。默认情况下 dnsmasq.conf 中只开启了最后 include 项，因此可以在 /etc/dnsmasq.conf 的前提下，将自定义的配置放到 /etc/dnsmasq.d 目录下的一个任意名字的配置文件当中。

>注意： /etc/dnsmasq.d/*.conf 的优先级大于 /etc/dnsmasq.conf

关于 dnsmasq 的配置项非常多，具体配置项含义在 `/etc/dnsmasq.conf` 中有详细的说明，本文如下配置实现一个简单的 DNS 服务器（配置文件放到了 `/etc/dnsmasq.d/` 目录下，命名为 `dnsmasq.conf`）：
```bash
#dnsmasq 启动监听的端口号
port=53

#从不转发格式错误的域名
domain-needed

#默认情况下Dnsmasq会发送查询到它的任何上游DNS服务器上，如果取消注释，
#则Dnsmasq则会严格按照/etc/resolv.conf中的 DNS Server 顺序进
#行查询，直到第一个成功解析成功为止。
strict-order

# dnsmasq 缓存大小，默认 150
cache-size=8192

#address 可以将指定的域解析为一个IP地址，即泛域名解析。
# 将 *.taobao.com 解析到 10.10.10.10
address=/taobao.com/10.10.10.10

#把所有.cn的域名全部通过 114.114.114.114 这台国内DNS服务器来解析
server=/cn/114.114.114.114
```
为了验证 /etc/hosts 文件解析是否起作用，我们也向 hosts 文件添加几条记录：
```bash
10.4.29.106      ansible
10.4.24.116      www.baidu.com
```

>注意：/etc/hosts 文件修改后需要重启 dnsmasq，否则修改不会生效。
重启方法：`systemctl restart dnsmasq`

#### 启动
```bash
# 设置为开机自启动
systemctl enable dnsmasq
# 启动 dnsmasq 服务
systemctl start dnsmasq
```

### 测试使用 Dnsmasq
我们搭建的 DNS 服务器地址为：`192.168.10.200`

使用 dig 命令指定 DNS 服务器地址来查看解析是否生效：
```bash
dig @192.168.10.200 ansible
dig @192.168.10.200 www.taobao.com
dig @192.168.10.200 ip.cn
```

### 验证 Dnsmasq 缓存功能是否生效
首先使用 dig 查询一个之前未查询过的域名，然后看响应时间是多少：
第一次 dig：
```bash
dig @192.168.10.200 qhh.me
......                                                                                                                                
;; Query time: 478 msec
;; SERVER: 192.168.10.200#53(192.168.10.200)
;; WHEN: Sat Apr 27 21:45:24 CST 2019
;; MSG SIZE  rcvd: 56
```
第二次 dig：
```bash
dig @192.168.10.200 qhh.me                                                                                                                                   
......
;; Query time: 0 msec
;; SERVER: 192.168.10.200#53(192.168.10.200)
;; WHEN: Sat Apr 27 21:45:32 CST 2019
;; MSG SIZE  rcvd: 67
```
可以看到两次同样的 dig 查询的时间不一样，第一次 478 ms，第二次 0 ms，说明第二次直接是从缓存中取的数据，没有向上游服务器发起请求。

### Dnsmasq 的缓存在哪里？如何查看？
dnsmasq 的缓存并不是保存在本地磁盘的某个文件，而是存储在内存中，因此是无法直接查看的。当然作为一个 Geek，想要查看缓存的内容也是有办法的：
1. dnsmasq 启动参数添加 --log-queries
```bash
vi /usr/lib/systemd/system/dnsmasq.service
ExecStart=/usr/sbin/dnsmasq -k 改为：ExecStart=/usr/sbin/dnsmasq -k --log-queries
```
2. 重新加载 Systemd Unit 配置文件
```
systemctl daemon-reload
```
3. 重启 dnsmasq
```
systemctl restart dnsmasq
```
4. 执行如下命令 dump 出来缓存内容到 journal 日志
```bash
kill -SIGUSR1 <PID>
```
5. 查看 dump 出来的 dns 记录（dnsmasq 当前缓存的内容）
```bash
journalctl -u dnsmasq
```

### 参考资料
http://www.thekelleys.org.uk/dnsmasq/doc.html | dnsmasq 官方文档
https://www.hi-linux.com/posts/30947.html | 一篇比较全面的博客
https://yq.aliyun.com/articles/582537 | 一篇比较精简的博客
http://flux242.blogspot.com/2012/06/dnsmasq-cache-size-tuning.html | 介绍了 dnsmasq 的基本概念、缓存淘汰机制等相关内容

