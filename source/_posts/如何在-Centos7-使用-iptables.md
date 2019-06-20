---
title: 如何在 Centos7 使用 iptables
date: 2019-06-09 20:54:15
categories: Linux
tags: Linux
---

![Alt text](/images/centos-iptables.png)

从 Red Hat Enterprise Linux (RHEL) 7 和 CentOS 7 开始，[firewalld](https://firewalld.org/) 便作为系统默认的防火墙软件，替代之前的 iptables。firewalld 使用 `firewall-cmd` 命令管理防火墙规则，但是对于习惯了 iptables 的用户来说更倾向于使用传统的 iptables 方式，因为没有学习成本，能马上使用。即便 iptables 不再是 RHEL 7 和 CentOS 7 默认的防火墙管理软件，但是它们并没有完全屏蔽 iptables，通过安装依然可以使用。

#### 简单说下 iptables 和 firewalld 区别
- [firewalld](https://firewalld.org/)  是 Red Hat Enterprise Linux (RHEL) 7 和 CentOS 7 开始开始引进的防火墙管理软件；
- firewalld 可以动态修改单条规则，而不需要像iptables那样，在修改了规则后必须得全部刷新才可以生效；
- firewalld 在使用上要比 iptables 人性化很多，即使不明白“五张表五条链”而且对 TCP/IP 协议不理解也可以实现大部分功能；
- firewalld 需要每个服务都去设置才能放行，因为默认是拒绝。而 iptables 里默认是每个服务是允许，需要拒绝的才去限制；
- firewalld 自身并不具备防火墙的功能，而是和 iptables 一样需要通过内核的 [netfilter](https://www.netfilter.org/) 来实现，也就是说 firewalld 和 iptables一样，他们的作用都是用于维护规则，而真正使用规则干活的是内核的 netfilter，只不过 firewalld 和 iptables 的结构以及使用方法不一样罢了；
- firewalld 底层调用的命令仍然是 iptables；
下图是 iptables 和 firewalld 的关系:
![Alt text](/images/iptables-firewalld.png)


---
下面我们介绍下如何在 Centos7 系统下继续使用传统的 iptables 来管理防火墙规则。

#### 关闭并注销 systemd 管理的 firewalld 服务
```
$ systemctl stop firewalld
# 注销 systmed 管理服务的过程相当于将原先指向相应 service 的软件链接
# 重新指向了 /dev/null
$ systemctl mask firewalld
```
#### 安装并配置 iptables
1. 安装 iptables 命令行工具
```bash
$ yum install -y iptables
```
2. 安装 iptables 服务（安装后默认归 systemd 管理）
```bash
$ yum install -y iptables-services
```
	这一步`iptables-services`安装完后会自动生成 iptables 的规则文件:  `/etc/sysconfig/iptables`
3. 设置开机自启动
```bash
$ systemctl enable iptables
```
4. 启动 iptables 防火墙服务
启动防火墙后即可用 `iptables -L -n` 命令查看当前防火墙规则了。
```bash
$ systemctl start iptables
$ iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
```
5. iptables 添加防火墙规则，有两种方法：
  - 通过 iptables 命令行工具添加：`iptables -I INPUT ...`；
  这种方式添加的规则不需要重启 iptables 即可立即生效，但是配置在重启服务器后会丢失，想持久保存下来执行如下命令（该命令会自动把命令行配置的规则写入 `/etc/sysconfig/iptables`，从而实现 iptables 规则的持久化保存）：
  ```bash
  $ service iptables save
  ```
	  ⚠️注意 `service iptables save` 和 `iptables-save` 命令的区别：
	- `service iptables save` 作用是将 `iptables` 命令编辑的防火墙规则持久化保存下来，保存到 `/etc/sysconfig/iptables`；
	- `iptables-save` 的作用是将内核中当前存在的防火墙规则导出来，和直接 `cat /etc/sysconfig/iptables` 的效果是一样的；
  - 通过编辑 `/etc/sysconfig/iptables` 文件添加防火墙规则；
	通过这种方式添加的防火墙规则，需要重启 iptables 服务才能生效，并且服务器重启后配置的规则依然保留：
	```bash
	$ systemctl restart iptables
	```
	举例：添加/删除 禁止 ping 响应的规则
	```bash
	添加禁止 ICMP 回显响应规则：
	iptables -A INPUT -i eth1 -p icmp -m icmp --icmp-type 8 -j DROP
	删除上面规则：
	iptables -D INPUT -i eth1 -p icmp -m icmp --icmp-type 8 -j DROP
	```
6. 查看防火墙状态
```bash
$ systemctl status iptables
```

#### 相关资料
http://shaozhuqing.com/?p=4787 | iptables 和 firewalld 关系
https://blog.51cto.com/xjsunjie/1902993 | 细说firewalld和iptables
https://support.rackspace.com/how-to/use-iptables-with-centos-7/ | Use iptables with CentOS 7
https://o-my-chenjian.com/2017/02/28/Using-Iptables-On-Centos7/ | 在Centos7上使用Iptables

