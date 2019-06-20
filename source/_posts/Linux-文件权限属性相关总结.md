---
title: Linux 文件权限属性相关总结
date: 2018-11-18 19:35:10
categories: Linux
tags: Linux
---

## Linux 文件权限属性相关总结
### 基础知识
![](/images/linux_file_attr_pem.png)
### 文件权限与属性修改
- chgrp: 更改文件属组
- chown: 更改文件属主
- chmod: 更改文件权限，SUID、SGID、SBIT 等属性

#### 1. 更改文件属组
更改时组名必须存在，即在必须在 `/etc/group` 文件内存在，否则报错。
```bash
命令格式: chgrp [-R] group_name dirname/filename ...
选顷参数:
-R : 递归(recursive)更改，即连同子目彔下的所有文件、目录
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant vagrant 5 Nov 17 19:04 aa
[root@centos7 vagrant]# chgrp root aa
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant root 5 Nov 17 19:04 aa
```

#### 2. 更改文件属主
```bash
命令格式: 
chown [-R] 账号名称 文件或目彔（只更改属主）
chown [-R] 账号名称:组名 文件或目彔（属主和属组同时更改）
选顷参数:
-R : 递归(recursive)更改，即连同子目彔下的所有文件、目录
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant root 5 Nov 17 19:04 aa
[root@centos7 vagrant]# chown root aa
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 root root 5 Nov 17 19:04 aa
[root@centos7 vagrant]# chown vagrant:vagrant aa
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant vagrant 5 Nov 17 19:04 aa
```
**Tips:**
`chown` 也能修改属组：`chown .group_name filename`
```bash
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant vagrant 5 Nov 17 19:04 aa
[root@centos7 vagrant]# chown .root aa
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant root 5 Nov 17 19:04 aa
```

#### 3. 更改文件权限
更改文件权限使用 chmod 命令，该命令有两种使用方式：以数字或者符号来进行权限的变更。
- 数字方式更改
```
命令格式: chmod [-R] xyz 文件或目录
选项参数：
-R: 递归(recursive)更改，即连同子目彔下的所有文件、目录
xyz: 权限数字
[root@centos7 vagrant]# ls -l
total 4
-rw-rw-r--. 1 vagrant root 5 Nov 17 19:04 aa
[root@centos7 vagrant]# chmod 755 aa
[root@centos7 vagrant]# ls -l
total 4
-rwxr-xr-x. 1 vagrant root 5 Nov 17 19:04 aa
```
- 符号方式更改
![](/images/linux_chmod.png)
```bash
将文件权限设置为: -rwxr-xr-x
[root@centos7 vagrant]# chmod u=rwx,g=rx,o=rx aa
给所有人赋予文件可写权限
[root@centos7 vagrant]# chmod a+w aa
所在组和其他组人去除可写权限
[root@centos7 vagrant]# chmod g-x,o-x aa
```

### 文件与目录的权限区别
Linux 下文件与目录的权限（r、w、x）有很大的不同，具体如下：
![](/images/linux_file_dir_pem.png)

### Linux FHS 标准
Linux FHS（[Filesystem Hierarchy Standard](http://www.pathname.com/fhs/)）文一种规范，规范 Linux 各发行版的目录结构，什么目录下该存放什么类型的文件。大概规范如下（其中灰色部分目录不能在系统的不同磁盘设备，因为都是和系统启动有关的，必须在系统盘所在的磁盘）：

![](/images/linux_fhs.png)

