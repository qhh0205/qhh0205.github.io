---
title: 使用 Stunnel 隐藏 OpenVPN 流量
date: 2019-06-23 09:59:30
categories: OpenVPN
tags: OpenVPN
---

### 简介
众所周知的原因，在海外直接搭建 [OpenVPN](https://openvpn.net) 根本无法使用（TCP 模式），或者用段时间就被墙了（UDP 模式）。本文主要介绍如何通过 [Stunnel](https://www.stunnel.org/) 隐藏 OpenVPN 流量，使其看起来像普通的 SSL 协议传输，从而绕过 gfw。



Stunnel 分为客户端和服务端，客户端负责接收用户 OpenVPN 客户端流量并转化成 SSL 协议加密数据包，然后转发给 Stunnel 服务端，实现 SSL 协议数据传输，服务端然后将流量转化成 OpenVPN 流量传输给 OpenVPN 服务端。因此我们可以在国内搭 Stunnel 客户端，国外搭 Stunnel 服务端。OpenVPN + Stunnel 整体架构如下：

![Alt text](/images/stunnel-openvpn.png)


### Stunnel 隐藏 OpenVPN 流量具体过程
#### 1. 首先需要有个 OpenVPN 服务端
关于 OpenVPN 的搭建及使用在这里不多说了，之前写过文章，详情见[这里](https://qhh.me/2019/06/16/Cenos7-%E4%B8%8B%E6%90%AD%E5%BB%BA-OpenVPN-%E8%BF%87%E7%A8%8B%E8%AE%B0%E5%BD%95/)。这里要说明的是，Stunnel 不支持 udp 流量转换，所以 OpenVPN 需要以 TCP 模式运行。下面为 OpenVPN TCP 模式的配置示例：
```
port 4001   # 监听的端口号
proto tcp-server
dev tun
ca /etc/openvpn/server/certs/ca.crt  #   CA 根证书路径
cert /etc/openvpn/server/certs/server.crt  # open VPN 服务器证书路径
key /etc/openvpn/server/certs/server.key  # open VPN 服务器密钥路径，This file should be kept secret
dh /etc/openvpn/server/certs/dh.pem  # Diffie-Hellman 算法密钥文件路径
tls-auth /etc/openvpn/server/certs/ta.key 0 #  tls-auth key，参数 0 可以省略，如果不省略，那么客户端
# 配置相应的参数该配成 1。如果省略，那么客户端不需要 tls-auth 配置
server 10.8.0.0 255.255.255.0   # 该网段为 open VPN 虚拟网卡网段，不要和内网网段冲突即可。open VPN 默认为 10.8.0.0/24
push "dhcp-option DNS 8.8.8.8"  # DNS 服务器配置，可以根据需要指定其他 ns
push "dhcp-option DNS 8.8.4.4"
push "redirect-gateway def1"   # 客户端所有流量都通过 open VPN 转发，类似于代理开全局
compress lzo
duplicate-cn   # 允许一个用户多个终端连接
keepalive 10 120
comp-lzo
persist-key
persist-tun
user openvpn  # open VPN 进程启动用户，openvpn 用户在安装完 openvpn 后就自动生成了
group openvpn
log /var/log/openvpn/server.log  # 指定 log 文件位置
log-append /var/log/openvpn/server.log
status /var/log/openvpn/status.log
verb 3
```

#### 2. Stunnel 服务端安装配置
##### 安装配置 Stunnel 服务端（海外节点）：
```
yum -y install stunnel
cd /etc/stunnel
openssl req -new -x509 -days 3650 -nodes -out stunnel.pem -keyout stunnel.pem
chmod 600 /etc/stunnel/stunnel.pem
vim stunnel.conf 填入如下内容:
pid = /var/run/stunnel.pid
output = /var/log/stunnel.log
client = no
[openvpn]
accept = 443   
connect = 127.0.0.1:4001
cert = /etc/stunnel/stunnel.pem
```
说明：
>accept = 443   # Stunnel 服务端监听端口
connect = 127.0.0.1:4001  # OpenVPN 服务端地址

##### 使用 systemd  启动 Stunnel 服务端：
为了管理方便，我们使用 systemd 管理 Stunnel 服务，编辑一个 systemd unit 文件，vim /lib/systemd/system/stunnel.service：
```ini
[Unit]
Description=SSL tunnel for network daemons
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target
Alias=stunnel.target

[Service]
Type=forking
ExecStart=/usr/bin/stunnel /etc/stunnel/stunnel.conf
ExecStop=/usr/bin/killall -9 stunnel

# Give up if ping don't get an answer
TimeoutSec=600

Restart=always
PrivateTmp=false
```
启动 Stunnel 服务端：
```
systemctl start stunnel.service
systemctl enable stunnel.service
```

#### 3. Stunnel 客户端安装配置
Stunnel 的客户端安装和服务器一样，同样的软件，既可以作为客户端，也可以作为服务端，只是配置不同而已。
##### 安装配置 Stunnel 客户端（国内节点）：
```
yum -y install stunnel
cd /etc/stunnel
scp ....  # 将服务端的证书 stunnel.pem 拷贝到这里
chmod 600 /etc/stunnel/stunnel.pem
vim stunnel.conf 填入如下内容：
pid=/var/run/stunnel.pid
output=/var/log/stunnel.log
client = yes

[openvpn]
accept=8443
connect=stunnel_server_ip:443
cert = /etc/stunnel/stunnel.pem
```
说明：
>accept=8443  # Stunnel 客户端监听端口
>stunnel_server_ip:443 # stunnel 服务端 ip 及端口

##### 使用 systemd  启动 Stunnel 客户端：
这里前面同服务端的操作过程，不再赘述。
启动 Stunnel 客户端：
```
systemctl start stunnel.service
systemctl enable stunnel.service
```
#### 4. 使用 OpenVPN 连接 Stunnel
Stunnel + OpenVPN 都配好后，就可以使用 OpenVPN 客户端实现自由上网了，需要注意的是 OpenVPN 客户端现在需要连接的是 Stunnel 客户端，不再是直接连接 OpenVPN 服务端。

### 相关文档
https://github.com/Xaqron/stunnel
