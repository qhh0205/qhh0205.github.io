---
title: Linux 进程树查看工具 pstree
date: 2019-02-16 19:39:42
categories: Linux
tags: Linux
---

### 简介
[pstree](http://psmisc.sourceforge.net/) 是 Linux 下的一个用于展示进程树结构的工具，类似于 [tree](https://linux.die.net/man/1/tree) 展示目录树一样，可视化地查看进程的继承关系。pstree 工具其实是 [PSmisc 工具集](http://psmisc.sourceforge.net/)的成员之一，PSmisc 工具集由 4 个实用的 Linux 进程管理工具（通过 Linux 的 /proc 文件系统实现）组成：
- *fuser* - identifies what processes are using files.
- *killall* - kills a process by its name, similar to a pkill found in some other Unices.
- *pstree* - Shows currently running processes in a tree format.
- *peekfd* - Peek at file descriptors of running processes.

>**pstree 带来的方便之处:**
一条命令就可以很轻松地追溯某个进程的继承关系，再也不需要通过多次执行 `ps -ef` 一级一级的查看进程的继承关系。

### 安装
#### On Fedora/Red Hat/CentOS
```bash
sudo yum install -y psmisc
```

#### On Mac OS
```bash
brew install pstree
```
#### On Ubuntu/Debian APT
```bash
sudo apt-get install psmisc
```

### 使用
#### 语法
`pstree [选项]`

#### 选项
>-a：显示每个程序的完整指令，包含路径，参数或是常驻服务的标示；
-c：不使用精简标示法；
-G：使用VT100终端机的列绘图字符；
-h：列出树状图时，特别标明现在执行的程序；
-H<程序识别码>：此参数的效果和指定"-h"参数类似，但特别标明指定的程序；
-l：采用长列格式显示树状图；
-n：用程序识别码排序。预设是以程序名称来排序；
-p：显示程序识别码；
-u：显示用户名称；
-U：使用UTF-8列绘图字符；
-V：显示版本信息。

#### 示例
1. 显示 PID 为 2858 进程的进程树;
```bash
[vagrant@docker ~]$ pstree 2858
dockerd─┬─2*[docker-proxy───4*[{docker-proxy}]]
        └─9*[{dockerd}]
```
2. 显示 PID 为 2858 进程的进程树，同时列出每个进程的 pid;
*注意：可以观察出，大括号括起来的为线程！*
```bash
[vagrant@docker ~]$ pstree -p 2858
dockerd(2858)─┬─docker-proxy(4378)─┬─{docker-proxy}(4379)
              │                    ├─{docker-proxy}(4380)
              │                    ├─{docker-proxy}(4381)
              │                    └─{docker-proxy}(4382)
              ├─docker-proxy(6582)─┬─{docker-proxy}(6583)
              │                    ├─{docker-proxy}(6585)
              │                    ├─{docker-proxy}(6586)
              │                    └─{docker-proxy}(6587)
              ├─{dockerd}(2997)
              ├─{dockerd}(2998)
              ├─{dockerd}(2999)
              ├─{dockerd}(3000)
              ├─{dockerd}(3222)
              ├─{dockerd}(3223)
              ├─{dockerd}(3224)
              ├─{dockerd}(4480)
              └─{dockerd}(4493)
```

3. 显示 PID 为 2858 进程的进程树，同时列出每个进程的 pid 和启动进程的命令行;
```bash
[vagrant@docker ~]$ pstree -p 2858 -a
dockerd,2858 -H fd://
  ├─docker-proxy,4378 -proto tcp -host-ip 0.0.0.0 -host-port 3306 -container-ip 172.17.0.2 -container-port 3306
  │   ├─{docker-proxy},4379
  │   ├─{docker-proxy},4380
  │   ├─{docker-proxy},4381
  │   └─{docker-proxy},4382
  ├─docker-proxy,6582 -proto tcp -host-ip 0.0.0.0 -host-port 8080 -container-ip 172.17.0.3 -container-port 80
  │   ├─{docker-proxy},6583
  │   ├─{docker-proxy},6585
  │   ├─{docker-proxy},6586
  │   └─{docker-proxy},6587
  ├─{dockerd},2997
  ├─{dockerd},2998
  ├─{dockerd},2999
  ├─{dockerd},3000
  ├─{dockerd},3222
  ├─{dockerd},3223
  ├─{dockerd},3224
  ├─{dockerd},4480
  └─{dockerd},4493
```

4. 直接执行 `pstree` 默认列出整个系统的进程树;

### 相关资料
http://man.linuxde.net/pstree
http://psmisc.sourceforge.net
https://www.wikiwand.com/en/Pstree
