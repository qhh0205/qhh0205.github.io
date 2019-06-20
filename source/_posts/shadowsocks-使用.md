---
title: shadowsocks 使用
date: 2018-03-18 09:27:47
categories: vpn
tags: shadowsocks
---

### 安装
[https://github.com/shadowsocks/shadowsocks/blob/master/README.md](https://github.com/shadowsocks/shadowsocks/blob/master/README.md)
### 命令行启动
- 前台启动
``sudo ssserver -p 443 -k password -m aes-256-cfb``
- 后台启动
``sudo ssserver -p 443 -k password -m aes-256-cfb -d start``
- 停止
``sudo ssserver -d stop``
- 查看日志
``sudo less /var/log/shadowsocks.log``

### 使用配置文件启动（墙裂推荐该方式）
- 前台启动：
``sudo ssserver -c /etc/shadowsocks.json``

- 后台启动：
``sudo ssserver -c /etc/shadowsocks.json -d start``
- 停止服务：
``sudo ssserver -c /etc/shadowsocks.json -d stop``

[shadowsocks.json 配置文件格式](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)

配置文件示例（单服务器多账号配置）:
```json
{
    "server":"0.0.0.0",
    "local_address": "127.0.0.1",
    "local_port：":1080,
    "port_password":{
      "1851":"xxxx",
      "1852":"xxxx",
      "1853":"xxxx",
      "1854":"xxxx",
      "1855":"xxxx",
      "1856":"xxxx"
    },
    "timeout":300,
    "method":"aes-256-cfb",
    "fast_open": false
}
```

### 故障排查
如果 shadowsocks.json 配置文件中 server 字段配成公网IP，而不是 ``0.0.0.0``，会报如下错误：

```socket.error: [Errno 99] Cannot assign requested address```
