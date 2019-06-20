---
title: 配置 gcloud 使用 Shadowsocks HTTP 代理
date: 2018-04-14 17:06:55
categories: Google Cloud Platform
tags: gcp
---
## 配置 gcloud 使用 Shadowsocks HTTP 代理
### 1. 问题描述
最近在使用 [gcloud](https://cloud.google.com/sdk/?hl=zh-cn) 访问 [gcp(Google Cloud Platform)](https://cloud.google.com/?hl=zh-cn) 的资源，但是由于 GFW 的原因必须得配个 HTTP 代理才能访问。虽然之前装个 Shadowsock-X 可以突破 GFW 用浏览器访问 Google，但是 Shadowsock-X 默认只开启 SOCKS 代理，并没有提供 HTTP 代理。为了让 shadowsocks 开启 HTTP 代理，必须得想一些办法了，要不然没法工作了。。。
### 2. 解决问题
经过一番 google，取而代之的是使用 [Shadowsocks-NG](https://github.com/shadowsocks/ShadowsocksX-NG/releases) 客户端，其实就是 [Shadowsocks-X](https://github.com/shadowsocks/shadowsocks/wiki/Ports-and-Clients#os-x) 的升级版，有了更多丰富的功能，最主要的是该客户端启动后默认就开启了 HTTP 代理，可以直接供 gcloud 等命令行工具使用。具体配置 gcloud 使用 Shadowsocks-NG http 代理的方法如下：
1. 点击如下链接下载并安装 Shadowsocks-NG：
https://github.com/shadowsocks/ShadowsocksX-NG/releases
2. 启动 Shadowsocks-NG，填入 shadowsocks 服务端 ip，端口，加密方式等信息
注意：不能勾选`启用 OTA（被启用）`复选框
![](/images/ss-1.png)

3. 获取 HTTP 代理的 IP 和端口
点击`偏好设置`查看 HTTP 代理 IP 及端口：
![](/images/ss-2.png)
![](/images/ss-3.png)
4. 设置 gcloud HTTP 代理
使用如下命令设置上一步获取的 Shadowsocks-NG HTTP 代理：
``` bash
gcloud config set proxy/type http
gcloud config set proxy/address 127.0.0.1
gcloud config set proxy/port 1087
```
5. 接下来就可以畅通无阻地访问 google 云平台资源了

### 3. 相关链接
https://www.stefanwienert.de/blog/2018/01/21/shadowsocks-quick-guide-for-restricted-internet-environments/

