---
title: shell多进程并发编程
date: 2017-12-03 21:27:19
categories:
    - Shell脚本
tags: Shell
---
在shell中使用多进程并发处理是非常方便的，如果有一个定时任务是同时ping主机ip列表，检测目标机器是否down掉，就可以用shell的多进程来实现，同时ping多个主机，不影响定时任务的执行。shell的实现方式是通过 & 符号来使要执行的进程后台执行，然后主调shell通过wait来等待所有后台执行完毕，然后退出主调shell。以下是一个心跳检测脚本，通过ping来批量检测机器是否可达，如果不可达则启用备用机器的服务，并发送微信报警。
``` bash
#!/bin/bash
 
cd ~/scripts
# $1:host $2:server_type $3:port $4=start_server
#  将要执行的过程封装成函数
function heartbeat_detection() {
    ping_loss=`ping -c 6 $1 | grep "100% packet loss"`
    Date=`date +"%Y-%m-%d %H:%M:%S"`
    if [ -z "$ping_loss" ]; then
        echo -e "Date: $Date|Host: $1|Monitor: Ping is ok." >> monitor_result.log
    else
        echo -e "Date: $Date|Host: $1|Problem: Unreachable for 2 minutes..." >> monitor_result.log
        python wechat_alert/wechat_alert.py "@all" "`Date: date +%F %H:%M:%S` Host:$1 Server_type:$2 Status:Down..."
        service_port=`netstat -lnp | grep $3 | awk '{print $NF}' | awk -F '/' '{print $1}'`
        if [ -z $service_port ]; then
            sh $4
        fi
    fi
}
 
for item in $(cat srv_list)
do
     host=`echo $item | awk -F '|' '{print $1}'`
     server_type=`echo $item | awk -F '|' '{print $2}'`
     port=`echo $item | awk -F '|' '{print $3}'`
     start_server=`echo $item | awk -F '|' '{print $4}'`
     #   后台执行任务(非阻塞)
     heartbeat_detection $host $server_type $port $start_server &
done
# 等待所有后台任务完成(阻塞)
wait
Date=`date +"%Y-%m-%d %H:%M:%S"`
echo -e "Date: $Date|Heartbeta detection finished..."
```
