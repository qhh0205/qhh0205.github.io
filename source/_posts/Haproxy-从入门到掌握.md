---
title: Haproxy 从入门到掌握
date: 2019-06-17 09:54:54
categories: HAProxy
tags: HAProxy
---

### 简介
[HAProxy](http://www.haproxy.org/) 是一款开源且免费的反向代理软件，为基于 TCP 和 HTTP 的应用提供高可用、负载均衡和代理功能。它特别适用于流量非常大的网站，为世界上访问量最大的网站提供了强大的支持。多年来，HAProxy 已经成为事实上的标准开源负载均衡器，大多数主流的 Linux 发行版已经自带了该安装包，并且在云平台也经常被使用。

HAProxy 是一个纯粹的反向代理软件，与 nginx 不同的是 haproxy 没有 web 服务功能，而且同时支持 4 层和 7 层代理。Nginx 从 1.9.0 才开始支持 4 层代理，通过 stream 模块支持，该模块默认不会自带安装，需要编译安装的时候手动添加上这个模块。
![Alt text](/images/haproxy.png)

HAProxy的核心功能：
- 负载均衡：L4 和 L7 两种模式，支持 RR/静态RR/LC/IP Hash/URI Hash/URL_PARAM Hash/HTTP_HEADER Hash等丰富的负载均衡算法；
- 健康检查：支持 TCP 和 HTTP 两种健康检查模式；
- 会话保持：对于未实现会话共享的应用集群，可通过 Insert Cookie/Rewrite Cookie/Prefix Cookie，以及上述的多种 Hash 方式实现会话保持；
- SSL：HAProxy 可以解析 HTTPS 协议，并能够将请求解密为 HTTP 后向后端传输；
- HTTP请求重写与重定向；
- 监控与统计：HAProxy提供了基于 Web 的统计信息页面，展现健康状态和流量数据。基于此功能，使用者可以开发监控程序来监控 HAProxy 的状态；

### Centos7 下安装 HAProxy
```
wget http://www.haproxy.org/download/1.8/src/haproxy-1.8.20.tar.gz
tar zxvf haproxy-1.8.20.tar.gz
yum groupinstall -y 'Development Tools'  # 安装 gcc 相关软件
cd haproxy-1.8.20
make TARGET=linux2628  # 编译：TARGET 和内核版本有关，不同的内核版
                       # 本对应不同值，对应关系在 README
make install # 安装到系统路径
```

### 使用 HAProxy 搭建一个 L4 层代理
这里使用 HAProxy 转发流量到后台 3 个 Shadowsocks 节点的 1443/tcp 端口，并配有 TCP 健康检查机制:
![Alt text](/images/haproxy-arch.png)

HAProxy 的 L4 层代理配置很简单，定义一对 frontend 和 backend，frontend 为 haproxy 前端监听的端口，backend 为后端服务器节点，我们访问 haproxy 不同的端口即可访问到对应的后端服务。frontend 和 backend 通过 default_backend 后面的名称关联。其他的配置项说明见下面配置文件 haproxy.cfg 中注释说明：

新建一个 HAProxy 配置文件：
mkdir /etc/haproxy
vim /etc/haproxy/haproxy.cfg 填入如下内容:
haproxy.cfg:
```ini
#
# demo config for Proxy mode
#
# global 为全局配置项，主要和 haproxy 进程本身有关。defaults 为默认配置，当后面的 listen、frontend、banckend 等块没有再次指明相关配置时，会继承 defaults 的配置。

global
    maxconn         20000 #最大连接数
    ulimit-n        204800  #ulimit的数量限制
    log             127.0.0.1 local3
    user             haproxy
    group            haproxy
    chroot          /var/empty
    nbproc      4 #启动后运行的进程数量
    daemon        #以后台形式运行haproxy
    pidfile     /var/run/haproxy.pid
defaults
    log global
    mode tcp
    retries 3 #3次连接失败认为服务不可用，也可以在后面设置
    timeout connect 5s #连接超时
    timeout client 30s #客户端超时
    timeout server 30s #服务器端超时
    option redispatch
    option    nolinger
    no option dontlognull
    option    tcplog
    option log-separate-errors

listen admin_stats  #监控页面设置
    bind 0.0.0.0:26000 # 监控页面监听端口号
    bind-process 1
    mode http
    log 127.0.0.1 local3 err
    stats refresh 30s ##每隔30秒自动刷新监控页面
    stats uri /admin
    stats realm welcome login\ Haproxy
    stats auth admin:123456
    stats hide-version
    stats admin if TRUE

frontend shadowsocks
  bind *:1443
  default_backend shadowsocks
backend shadowsocks
  # balance roundrobin #负载均衡的方式，roundrobin是轮询
  # check inter 1500 心跳检测频率, rise 3 是3次正确认为服务器可用，fall 3是3次失败认为服务器不可用
  server ss-node-1 192.168.0.1:1443 check inter 1500 rise 3 fall 3
  server ss-node-1 192.168.0.2:1443 check inter 1500 rise 3 fall 3
  server ss-node-1 192.168.0.3:1443 check inter 1500 rise 3 fall 3
```
测试配置文件是否有效
```
haproxy -f /etc/haproxy/haproxy.cfg -c
```
启动 HAProy 服务：
```
haproxy -f /etc/haproxy/haproxy.cfg
```
我们还可以访问 HAProxy 自带的监控页面：上面我们配置的访问地址为 haproxy_ip:26000/admin，账号：admin，密码：123456。HAProxy 自带的监控页面特别好用，可以看到每个后端节点的流量使用情况、在线状态、可以随时将节点从后端集群中剔除或者改变状态。

![Alt text](/images/haproxy-monitor-ui.png)


### 相关文档
http://www.haproxy.org/ | HAProxy 官网
https://www.jianshu.com/p/c9f6d55288c0 | HAProxy从零开始到掌握
https://www.jianshu.com/p/17c2f87bb27f | 简述Haproxy常见的负载均衡调度算法及应用场景详解
https://www.serverlab.ca/tutorials/linux/network-services/how-to-configure-haproxy-health-checks/ | HAProxy 健康检查配置
