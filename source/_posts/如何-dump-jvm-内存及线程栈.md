---
title: 如何 dump jvm 内存及线程栈
date: 2018-05-19 23:28:46
categories: Java
tags: Jvm
---

目前很多企业的后台服务都是 java 服务，在故障出现时能及时 dump jvm 内存和线程栈对于故障的分析及定位是非常重要的。接下来介绍如何进行 dump 操作，并分享一个简单脚本实现服务器线程数超过一定阀值时自动 dump 线程数最高的 java 进程的内存及线程栈。
#### 1. dump jvm 内存
命令格式：
```bash
jmap -dump:format=b,file=dump_file_name pid
```
举例：dump pid 为 `4738` 的 java 进程的内存到 `app_mem_dump.bin` 文件
```bash
jmap -dump:format=b,file=app_mem_dump.bin 4738
```

#### 2. dump jvm 线程栈
命令格式：
```bash
jstack pid > dump_file_name
```
举例：dump pid 为 `4738` 的 java 进程的线程栈到 `app_thread_dump.txt` 文件
```bash
jstack 4738 > app_thread_dump.txt
```
### 脚本分享
当服务器线程数超过 2500 时自动 dump 线程数最高的 java 进程的内存及线程栈。
```bash
#!/usr/bin/env bash
# 
# 服务器线程数达到 2500 以上时 dump 线程数最多的 java 进程的线程及内存
#
source ~/.bashrc
cur_thread_num=`ps -efL | wc -l`
if [ $cur_thread_num -le 2500 ]; then
    exit 0
fi

cur_date=`date +"%Y-%m-%d_%H-%M-%S"`
cd ./dumpfile
# 服务器当前线程 dump 到文件:按照线程数由大到小排序显示
ps -efL --sort -nlwp > server_thread_dump_$cur_date
# dump 线程数最多的 jvm 的线程及内存
most_thread_num_pid=`cat server_thread_dump_$cur_date | sed -n '2p' | awk '{print $2}'`
nohup jstack -l $most_thread_num_pid > java_app_thread_dump_${cur_date}_pid_${most_thread_num_pid} &
nohup jmap -dump:format=b,file=java_app_mem_dump_${cur_date}_pid_${most_thread_num_pid} $most_thread_num_pid &

exit 0
```
