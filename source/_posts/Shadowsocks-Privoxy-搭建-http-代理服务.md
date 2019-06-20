---
title: Shadowsocks + Privoxy 搭建 http 代理服务
date: 2018-05-20 16:44:21
categories: Shadowsocks
tags: http 代理
---

目前很多软件都支持配置 http 代理来加速访问，或者绕过 GFW 来获取需要的资源。比如 docker、git、gcloud、curl 这些软件都支持 http 代理。那么我们该如何轻松地基于 shadowsocks 搭建一个 http 代理，其实很简单，使用 privoxy 代理软件将收到的 http 请求转发给 shadowsocks 客户端即可。

### Shadowsocks 软件整体架构

如下图所示，shadowsocks 由两部分组成：客户端（SS Local），服务端（SS Server）。客户端就是用来做本地 Sock5 代理的，代理本地 PC 的请求和服务端通信，我们一般在手机、平板、PC 上安装的图形化 shadowsokcs 软件就是 SS 客户端软件，当然如果在 Linux 下，也有SS 客户端：`sslocal`，后面会介绍到如何配合 `privoxy` 来实现 `http` 代理服务。服务端就是在 Linux 服务器上安装的 shadowsocks 服务端软件，供客户端连接，我们一般说的搭建 shadowsocks 代理就是在服务器上安装并配置 SS Server。
![Alt text](/images/ss-privoxy-http1.png)

### Shadowsocks + privoxy 搭建 http 代理服务步骤
整体架构如下图所示，我们需要找一台机器将 SS Server 搭建好，然后在局域网内的任何一台 Linux 服务器安装 SS Local 和 Privoxy，Privoxy 暴露 8118 端口作为 http 代理的端口：
![Alt text](/images/ss-privoxy-http2.png)

#### 1. 安装配置 Shadowsocks Server 端（ssserver）
Shadowsocks 是用 Python 编写的，因此可以通过如下命令直接安装（sslocal 和 ssserver 均已安装）：
```bash
sudo pip install shadowsocks
```
接下来编写 Shadowsocks Server 端的配置文件，配置监听端口，加密方式，密码等，新建 `/etc/shadowsocks.json` 文件，填入如下内容：
```json
{
    "server":"0.0.0.0", 
    "server_port":1851, # SS Server 端口
    "local_address": "127.0.0.1", #SS Local 端配置，不影响Server端使用
    "local_port":1080,  #SS Local 端配置，不影响Server端使用
    "password":"xxxxx",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
使用配置文件启动 SS Server：
```bash
ssserver -c /etc/shadowsocks.json -d start
```

#### 2. 安装配置 Shadowsocks 客户端（sslocal）
第一步已将 SS Server 安装并配置完成，服务端口为 `1851` ，接下来在需要安装 http 代理的机器上安装配置 shadowsocks 客户端，安装方法和第一步一样：
```bash
sudo pip install shadowsocks
```
编写 SS Local 客户端配置文件，配置远程连接 SS Server 的 IP，端口，密码，加密方式等，新建 `/etc/shadowsocks.json` 文件，填入如下内容：
```json
{
    "server":"xxx.xxx.xxx.xxx", # SS Server 端服务器公网 IP
    "server_port":1851, # SS Server 端口
    "local_address": "127.0.0.1", # SS Local 本地监听 IP
    "local_port：":1080, # SS Local 本地监听端口
    "password":"xxxxxx",
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```
使用配置文件启动 SS Local：
```bash
sslocal -c /etc/shadowsocks.json -d start
```
#### 3. 安装并配置 Privoxy
Privoxy 是一款代理软件，我们这里用该代理软件实现 HTTP 到 Socks5 的转换，所有来自 Privoxy 的请求被转发到 SS Local，从而实现了一个 HTTP 代理服务，Privoxy 的安装非常简单，直接 `yum` 一键搞定：
```bash
sudo yum install privoxy
```
编辑 Privoxy 配置文件 `/etc/privoxy/config`，搜索关键字 `listen-address` 找到 `listen-address 127.0.0.1:8118` 这一句，改成 `listen-address  0.0.0.0:8118`，表示该代理可以对外访问。
接下来在该配置该文件末尾添加 HTTP 请求转发到 SS Local Socks5 的配置：
```python
forward-socks5t / 127.0.0.1:1080 .
```
- `forward-socks5t`: 表示 Privoxy 转发请求到 Socks5 协议；
- `127.0.0.1`: 第二步中启动 SS Local 本地绑定 IP；
- `1080`: 第二步中启动 SS Local 本地监听端口；

启动 Privoxy：
```bash
systemctl restart privoxy
systemctl enable privoxy
```
#### 4. 测试代理是否可用
```bash
curl -x privoxy_ip:8118 https://www.google.com
```

### 参考文章
[https://vc2tea.com/whats-shadowsocks](https://vc2tea.com/whats-shadowsocks/)
[https://docs.lvrui.io/2016/12/12/Linux%E4%B8%AD%E4%BD%BF%E7%94%A8ShadowSocks-Privoxy%E4%BB%A3%E7%90%86](https://docs.lvrui.io/2016/12/12/Linux%E4%B8%AD%E4%BD%BF%E7%94%A8ShadowSocks-Privoxy%E4%BB%A3%E7%90%86/)
