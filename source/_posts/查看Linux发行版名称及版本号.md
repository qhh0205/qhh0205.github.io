---
title: 查看Linux发行版名称及版本号
date: 2018-01-01 18:58:13
categories:
    - Linux
tags: Linux
---
对于linx发行版及版本号的查看有如下几种方法，当一种方法失效的时候可以试试其他几种：
* cat /etc/os-release
```
[vagrant@k8s-master ~]$ cat /etc/os-release
NAME="CentOS Linux"  # 不带版本号且适合人类阅读的操作系统名称
VERSION="7 (Core)" # 操作系统的版本号。禁止包含操作系统名称，但是可以包含适合人类阅读的发行代号。
ID="centos" # 小写字母表示的操作系统名称，禁止包含任何版本信息。
ID_LIKE="rhel fedora" # 一系列空格分隔的字符串，此字段用于表明当前的操作系统 是从哪些"父发行版"派生而来。
VERSION_ID="7" # 小写字母表示的操作系统版本号，禁止包含操作系统名称与发行代号。
PRETTY_NAME="CentOS Linux 7 (Core)" # 适合人类阅读的比较恰当的发行版名称， 可选的包含发行代号与系统版本之类的信息，内容比较随意。
ANSI_COLOR="0;31" # 在控制台上显示操作系统名称的文字颜色。
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/" # 操作系统的主页地址， 或者特定于此版本操作系统的页面地址。
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"
```
* cat /etc/issue
```bash
[root@vps ~]# cat /etc/issue
\S
Kernel \r on an \m
```
* lsb_release -a
```bash
[root@vps ~]# lsb_release -a
LSB Version:	:core-4.1-amd64:core-4.1-noarch
Distributor ID:	CentOS
Description:	CentOS Linux release 7.3.1611 (Core) 
Release:	7.3.1611
Codename:	Core
```
*  cat /etc/redhat-release(针对redhat，Fedora)
```bash
[root@vps ~]# cat /etc/redhat-release
CentOS Linux release 7.3.1611 (Core)
```
* cat /proc/version
```bash
[root@vps ~]# cat /proc/version 
Linux version 3.10.0-514.26.2.el7.x86_64 (builder@kbuilder.dev.centos.org) (gcc version 4.8.5 20150623 (Red Hat 4.8.5-11) (GCC) ) #1 SMP Tue Jul 4 15:04:05 UTC 2017
```
参考文章：
[http://www.jinbuguo.com/systemd/os-release.html#](http://www.jinbuguo.com/systemd/os-release.html#)
[http://www.cnblogs.com/parrynee/archive/2010/05/16/1736652.html](http://www.cnblogs.com/parrynee/archive/2010/05/16/1736652.html)
