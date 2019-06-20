---
title: Zabbix通过进程名监控进程状态配置详解
date: 2018-02-18 22:03:43
categories: Zabbix
tags: Zabbix
---

有时候我们只能通过进程名监控一个进程是否停掉了，因为有的进程并没有对外提供端口号，以下记录了下详细步骤，通过这个示例会学到很多zabbix核心配置相关的东西。
总的来说，配置一个完整的监控流程如下：

1. 创建监控项，即配置要监控的指标，如内存的使用率，CPU的使用率，进程的运行状况等，配了监控项后就会定时收集机器的配置信息，然后等待zabbix server收集(zabbix agent被动模式)。
2. 创建触发器，触发器将监控项收集的数据通过触发器表达式进行评估。
在触发器表达式中我们可以定义哪些值范围是合理，哪些是不合理的，如果出现不合理的值，触发器会把状态改为PROBLEM，接下来就到了报警以及发邮件。
3. 创建动作，在zabbix中动作的意思是触发器触发后要进行的操作，一般是通过配置给相关负责人发送邮件，短信等通知。

下面配置监控服务器的logstash(开源实时日志同步项目)进程是否在运行：

1. 首先创建监控进程的监控项:
监控项的组成：key[参数]
![](/images/zabbix-proc1.png)
例如获取5分钟的负载情况：system.cpu.load[avg5]，avg5是对应的参数。
zabbix agent支持的所有key可以到这里找到：
[http://www.ttlsa.com/zabbix/zabbix-agent-types-and-all-keys/](http://www.ttlsa.com/zabbix/zabbix-agent-types-and-all-keys/)
在这里我们需要的是proc.num这个key，以下是对此key的详解：
![](/images/zabbix-proc2.png)
可以看到此监控项的返回值是进程数量，其中cmdline参数可以是进程名字包含的关键字，在这里我的进程的关键字是logstash，因此按如下方式创建监控logstash进程的监控项，表示机器所有用户所有状态的logstash进程数量：
![](/images/zabbix-proc3.png)

2. 创建对应监控项的触发器：
创建触发器主要是编写触发器表达式，也就是评估监控项是否在合理范围的表达式。触发器表达式格式如下：
<code>host:key[param].function(parameter)} operator constant
主机：监控项.函数(参数)} 表达式 常数</code>
对于触发器表达式更加详细的介绍请参考这里：
[http://www.ttlsa.com/zabbix/zabbix-trigger-expression/](http://www.ttlsa.com/zabbix/zabbix-trigger-expression/)
触发器表达式示例：
触发器名称：Processor load is too high on www.zabbix.com
```{www.zabbix.com:system.cpu.load[all,avg1].last(0)}>5```
触发器说明：
www.zabbix.com：host名称
system.cpu.load[all,avg1]：item值,一分内cpu平均负载值
last(0)：最新值
\>5：最新值大于5
如上所示，www.zabbix.com这个主机的监控项，最新的CPU负载值如果大于5，那么表达式会返回true，这样一来触发器状态就改变为“problem”了。
**在这里针对logstash进程触发器配置如下：**
![](/images/zabbix-proc4.png)
上面配置表示如果机器logstash进程数量的最新值小于1，就会触发报警。

3. 配置动作发送短信和邮件报警：
以下是短信配置方式，邮件配置类似，其中应用集是自己创建的，主要用来分类，具体的自行研究：
![](/images/zabbix-proc5.png)
![](/images/zabbix-proc6.png)

**参考文章：**
 [zabbix item key详解](http://www.ttlsa.com/zabbix/zabbix-item-key/)
 [zabbix agent 类型所有key](http://www.ttlsa.com/zabbix/zabbix-agent-types-and-all-keys/)
 [zabbix触发器表达式详解](http://www.ttlsa.com/zabbix/zabbix-trigger-expression/)
