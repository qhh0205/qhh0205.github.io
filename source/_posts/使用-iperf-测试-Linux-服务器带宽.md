---
title: 使用 iperf 测试 Linux 服务器带宽
date: 2019-07-02 18:49:34
categories: Linux
tags: Linux
---


### iperf 简介
[iperf](https://sourceforge.net/projects/iperf/) 是一个用于测试网络带宽的命令行工具，可以测试服务器的网络吞吐量。目前发现两个很实用的功能：
1. 测试服务器网络吞吐量：如果我们需要知道某台服务器的「最大」网络带宽，那么最好在同区域找两台同等配置的机器测试，因为带宽测试结果和两节点的距离有关、也和运营商的限制有关、也和服务器 CPU 核数有关。
2. 测试到服务端节点网速：如果我们想知道目前客户端到服务器的实际网速是多少，在服务器启动 iperf，客户端连接 iperf 服务端，测试结果就是当前客户端到服务器的真实网速。

### 工具安装
```bash
yum install -y iperf
```

### iperf 选项参数
#### 通用选项
```bash
-f <kmKM>    报告输出格式。 [kmKM]   format to report: Kbits, Mbits, KBytes, MBytes
-i <sec>    在周期性报告带宽之间暂停n秒。如周期是10s，则-i指定为2，则每隔2秒报告一次带宽测试情况,则共计报告5次
-p    设置服务端监听的端口，默认是5001
-u    使用UDP协议测试
-w n<K/M>   指定TCP窗口大小
-m    输出MTU大小
-M    设置MTU大小
-o <filename>    结果输出至文件
```

#### 服务端选项
```bash
-s    iperf服务器模式
-d    以后台模式运行服务端
-U    运行一个单一线程的UDP模式
```

#### 客户端选项
```bash
-b , --bandwidth n[KM] 指定客户端通过UDP协议发送数据的带宽（bit/s）该参数只对 udp 测试有效。默认是1Mbit/s
-c <ServerIP>  以客户端模式运行iperf，并且连接至服务端主机ServerIP。 eg:  iperf -c <server_ip>
-d    双向测试
-t    指定iperf带宽测试时间，默认是10s。  eg:  iperf -c <server_ip> -t 20
-P    指定客户端并发线程数，默认只运行一个线程。 eg,指定3个线程 : iperf -c <server_ip> -P 3
-T    指定TTL值
```

### 使用方法示例
准备两台服务器 A 和 B，并分别安装 iperf 命令行工具。
1. 测试 A 服务器的出站带宽：在 B 服务器启动 iperf 服务端，A 服务器使用 iperf 连接 B 服务
器 iperf 服务端，这样测试的就是 A 服务器的出口带宽：
```bash
B: iperf -s -i 2  # 启动服务端
A: iperf -c <B_server_ip> -i 2 -t 60  # 客户端链接
```

2. 测试 A 服务器的入站带宽：在 A 服务器启动 iperf 服务的，B 服务器使用 iperf 连接 A 服务器 iperf 服务端，这样测试的就是 A 服务器的入口带宽。
```bash
A: iperf -s -i 2  # 启动服务端
B: iperf -c <A_server_ip> -i 2 -t 60  # 客户端链接
```

### 测试结果示例
```
[root@com26-83 ~]# iperf -c x.x.x.x -i 2 -t 60
------------------------------------------------------------
Client connecting to x.x.x.x, TCP port 5001
TCP window size: 22.1 KByte (default)
------------------------------------------------------------
[  3] local 10.2.26.83 port 48234 connected with x.x.x.x port 5001
[ ID] Interval       Transfer     Bandwidth
[  3]  0.0- 2.0 sec   147 KBytes   603 Kbits/sec
[  3]  2.0- 4.0 sec   369 KBytes  1.51 Mbits/sec
[  3]  4.0- 6.0 sec   512 KBytes  2.10 Mbits/sec
[  3]  6.0- 8.0 sec   896 KBytes  3.67 Mbits/sec
[  3]  8.0-10.0 sec  1.62 MBytes  6.82 Mbits/sec
[  3] 10.0-12.0 sec  2.12 MBytes  8.91 Mbits/sec
[  3] 12.0-14.0 sec  3.38 MBytes  14.2 Mbits/sec
[  3] 14.0-16.0 sec  6.00 MBytes  25.2 Mbits/sec
[  3] 16.0-18.0 sec  8.00 MBytes  33.6 Mbits/sec
```

### 相关文档
https://www.cnblogs.com/zdz8207/p/linux-iperf.html
