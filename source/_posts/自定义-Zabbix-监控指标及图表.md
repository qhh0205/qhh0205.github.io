---
title: 自定义 Zabbix 监控指标及图表
date: 2018-05-19 09:11:05
categories: Zabbix
tags: Zabbix
---

### 问题描述
 有时候 Zabbix 监控系统的模版提供的监控指标并不能满足我们的需求，比如我们要监控服务器的线程数、TCP 连接数等，这些指标在 Zabbix 自带的模板中是没有的，这时候我们就需要自定义监控指标来实现可视化监控。本文以监控服务器的 TCP 连接数为例来说明如何自定义监控指标来实现可视化监控。

### 解决问题
**总体思路是：**
修改 Zabbix Agent 端的配置文件，添加监控指标的键值对 --> 重启 Zabbix Agent --> 在 Zabbix Server 端界面化控制台中的模板添加监控指标，指定配置文件中的键 --> 创建指标的可视化展示。
以下分步图文列出如何操作：
#### 1. 修改 Zabbix Agent 端配置文件，添加监控指标的键值对
`vim` 打开 Zabbix Agent 端配置文件 `/home/zabbix/zabbix/etc/zabbix_agentd.conf` ，末尾添加如下内容：
```bash
UnsafeUserParameters=1
UserParameter=tcp.num,netstat -atunp | grep ESTABLISHED | wc -l
```
- `UnsafeUserParameters`: 自定义指标必需要添加该行；
- `UserParameter`: 自定义指标的参数；
- `tcp.num`: 监控指标的键，在 Zabbix Server 端创建监控指标时会用到，可以随意命名，比如 `tcp.count`；
- `netstat -atunp | grep ESTABLISHED | wc -l`：监控指标的值（注意：该值必须是数值类型，否则报错），获取服务器的 TCP 连接数，键和值之间通过英文逗号分隔；

#### 2. 重启 Zabbix Agent 端
```bash
 /home/zabbix/zabbix/sbin/zabbix_agentd -c /home/zabbix/zabbix/etc/zabbix_agentd.conf
```

#### 3. 在 Zabbix Server 端界面化控制台创建监控指标
为了让所有监控机器都能生效，所以在这里选择 Zabbix Server 自带的系统模板 `Template OS Linux` 中添加指标：
![](/images/zbx_custom_metrix1.png)
![](/images/zbx_custom_metrix2.png)
![](/images/zbx_custom_metrix3.png)

#### 4. 创建指标的可视化展示
在第三步中已经完成了监控指标的创建，即 Zabbix Server 已经开始收集 Agent 端的数据，但是我们还没有配置相应指标的可视化图表展示，无法看到该指标随时间的推移的变化趋势，接下来我们创建一个梯度图来可视化展示该指标的变化:

![](/images/zbx_custom_metrix4.png)
![](/images/zbx_custom_metrix5.png)
![](/images/zbx_custom_metrix6.png)
![](/images/zbx_custom_metrix7.png)

到这里该指标的可视化图表已创建完成，可以跳转到控制台首页查看相应 Zabbix Agent 的图表：
![](/images/zbx_custom_metrix8.png)

### 注意事项
由于 `netstat` 命令在非 root 用户下使用会有警告信息：
```bash
[zabbix@awsuw7-189 ~]$ netstat -atunp | grep ESTABLISHED | wc -l
(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
329
```
这样 Zabbix Agent 配置文件中使用 `netstat -atunp | grep ESTABLISHED | wc -l` 获取到的 `value` 会是上面所有的输出（即字符串类型），而不是 `wc -l` 得到的数值类型，从而导致配置自定义监控指标后会报错：
![](/images/zbx_custom_metrix9.png)

解决方法其实比较简单，切换到 root 用户给 `netstat` 命令添加 `s` 权限即可解决：
```bash
[root@awsuw7-189 ~]# chmod u+s /bin/netstat
```
