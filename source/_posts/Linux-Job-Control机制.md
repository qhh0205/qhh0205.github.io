---
title: Linux Job Control机制
date: 2017-12-17 15:31:56
categories:
    - Linux
tags: Linux
---
#### 基本概念
在单一终端机下同时进行多个工作的行为管理称为Job Control。其实说的简单点就是在一个登录窗口下可以同时进行多个工作任务，比如我们在登录bash后，想要一边复制文件一边进行数据查找，一边进行编译，还可以一边进行vi程序编写。其实这些工作可以同时在一个shell中进行，这时就需要了解下linux的job control的使用。
#### 如何进行job control管理
那么如何进程linux job control管理，要进行job control管理，需要通过job控制命令来进行，下面一一介绍job control相关的命令：
1. **直接将命令丢到后台中执行的&：**
命令后面跟&符号可以将命令的执行丢到后台去执行，不需要等待当前命令结束后才可以继续执行其他命令，这样就可以实现在同一bash下进行多个任务了，注意这里所说的是丢到后台执行，不是在后台挂起。举例如下，将5个sleep程序丢到后台执行：
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[1] 22362
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[2] 22363
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[3] 22364
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[4] 22365
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[5] 22366
[root@iZj6c43t8c5urd16vueux0Z ~]# 
[1]   Done                    sleep 200
[2]   Done                    sleep 200
[3]   Done                    sleep 200
[4]-  Done                    sleep 200
[5]+  Done                    sleep 200
```
 可以看出每将一个任务丢到后台执行后，都会输出用中括号扩起来的job id以及该命令的PID，格式为：**[job number] PID**，在任务完成后会输出提示信息，表示该任务的完成状态，格式为：**[job number] 任务完成状态 任务命令行**，在上面输出中，后面两个任务的job id后面分别有个-和+这两个符号，+代表这个任务是最新添加的，-代表最近最后第二个被放置到后台的任务，如果超过最后第三个以后的任务，就不会有+/-符号的存在了，Done表示任务完成，sleep 200表示这个任务的命令行。
2. **将目前的工作丢到后台中暂停:[ctrl]-z：**
有时候我们可能需要将当前的工作暂时挂起(暂停执行)，需要在同一shell下临时处理其他工作。比如我正用vim编辑一个文件，突然想起要看下系统当前时间是几点了，这时就可以先将vim挂起到后台，然后执行date命令查看时间，完了后再将挂起的工作唤醒，继续工作。举例如下：
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# vim hello.sh  # 打开文件编辑,按ctrl+z将vim暂时挂起，执行其他命令

[1]+  Stopped                 vim hello.sh  # 表示已将vim挂到后台，处于暂停状态
[root@iZj6c43t8c5urd16vueux0Z ~]# date # 此时可以执行其他任何命令
Sun Dec 17 13:02:15 CST 2017
[root@iZj6c43t8c5urd16vueux0Z ~]# fg # 执行fg将最近添加到后台的任务唤到前台来执行，在此执行fg继续vim编辑，关于这个命令的使用后面会介绍
```
3. **查看目前的后台工作状态：jobs：**
如果想要知道目前有多少个工作在后台当中，就可以用jobs命令来查看。
>jobs:
  * -l 除了列出job number与命令串之外，同时列出PID的号码；
  * -r 仅列出正在后台run的工作；
  * -s 仅列出正在后台暂停(stop)的工作
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[1] 22610
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[2] 22611
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200
^Z
[3]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs
[1]   Running                 sleep 200 &
[2]-  Running                 sleep 200 &
[3]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs -l
[1]  22610 Running                 sleep 200 &
[2]- 22611 Running                 sleep 200 &
[3]+ 22612 Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs -r
[1]   Running                 sleep 200 &
[2]-  Running                 sleep 200 &
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs -s
[3]+  Stopped                 sleep 200
```
4. **将后台工作拿到前台来处理：fg：**
在前面的命令当中都是将job放到后台中处理，如果需要将后台中的job放到前台中执行，可以用fg命令。命令格式为：**fg %jobnumber**，注意：那个%是可选的。如果直接执行fg，后面不加jobnumber，则默认将最后放到后台的job调到前台，如果要指定要将需要的job调到前台，则可以先用jobs获取到jobnumber，然后fg指定需要的jobnumber即可将对应的job调到前台。
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[4] 22640
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[5] 22641
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200
^Z
[6]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs
[3]-  Stopped                 sleep 200
[4]   Running                 sleep 200 &
[5]   Running                 sleep 200 &
[6]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# fg 5 # 将5号job调到前台来执行
sleep 200
```
5. **让工作在后台下的状态变成运行中(Running):bg：**
通过前面的介绍我们知道ctrl+z可以将一个job在后台暂停，状态为Stopped，那么如何将这个暂停的后台job变成Running状态呢，同时该job还在后台。我们可以用bg命令实现，命令格式为:**bg %jobnumber**，注意：那个%是可选的。和fg类似，直接执行bg，后面不加jobnumber，则默认将最后放到后台的job的状态变为Running。
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 20
^Z         # 按下ctrl+z将该命令挂起到后台
[1]+  Stopped                 sleep 20
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 20
^Z         # 按下ctrl+z将该命令挂起到后台
[2]+  Stopped                 sleep 20
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs  # 查看当前后台job的状态
[1]-  Stopped                 sleep 20
[2]+  Stopped                 sleep 20
[root@iZj6c43t8c5urd16vueux0Z ~]# bg  # 将最近的添加到后台的job变为Running
[2]+ sleep 20 &
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs
[1]+  Stopped                 sleep 20
[2]-  Running                 sleep 20 &   #可以看出job号为2的job状态已经变为Running
```
6. **管理后台的job:kill：**
如果想要删除某个job或者将该job重新启动，可以用kill命令来实现，用kill来给job一个信号(signal)。命令格式为:**kill -signal %jobnumber**，注意这里是jobnumber，不是PID，当然用PID也可以删除job。
>kill: 
  * -l 列出目前kill能够使用的信号有哪些
  * -1 重新读取一次参数的配置文件(类似reload)
  * -2 代表由键盘输入ctrl+c同样的操作
  * -9 立刻强制删除一个job，一般用来终止一个异常程序
  * -15 以正常的程序方式终止一个job。
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[1] 22701  #后台Running
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200 &
[2] 22702  # 后台Running
[root@iZj6c43t8c5urd16vueux0Z ~]# sleep 200
^Z        # ctrl+z 后台暂停
[3]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs
[1]   Running                 sleep 200 &
[2]-  Running                 sleep 200 &
[3]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# kill -15 %3   # 注意，这里必须得加%，表示job号码，为了区分jobnumber和PID

[3]+  Stopped                 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs
[1]   Running                 sleep 200 &
[2]-  Running                 sleep 200 &
[3]+  Terminated              sleep 200   # 3号job已删除
[root@iZj6c43t8c5urd16vueux0Z ~]# jobs
[1]-  Running                 sleep 200 &
[2]+  Running                 sleep 200 &
```
7. **脱机管理问题：**
需要注意的是上面所说的将job放到后台并不是放到系统的后台去，只是放到当前shell环境的后台，在这种情况下，如果你以远程的连接方式连接到你的Linux主机，并且将job以&的方式放到后台去，当exit退出终端后，后台执行的job将会被中断。可以自己做个实验验证，在此不试验了。那么如何实现将job正真放到系统后台呢，即使exit退出终端job还可以继续工作。其实很简单，用nohup命令配合&即可，示例如下：
```bash
[root@iZj6c43t8c5urd16vueux0Z ~]# nohup sleep 200 &
[1] 22721
[root@iZj6c43t8c5urd16vueux0Z ~]# nohup: ignoring input and appending output to ‘nohup.out’

[root@iZj6c43t8c5urd16vueux0Z ~]# ps aux | grep sleep | grep -v grep
root     22721  0.0  0.0 107892   604 pts/1    S    15:21   0:00 sleep 200
[root@iZj6c43t8c5urd16vueux0Z ~]# exit  # 退出终端
logout
Connection to 47.91.229.209 closed.
# haohao @ MacBook-Pro-8 in ~ [15:23:15] 
$ sh hk.sh  # 重新登录, 查看进程是否还在
[root@iZj6c43t8c5urd16vueux0Z ~]#  ps aux | grep sleep | grep -v grep
root     22721  0.0  0.0 107892   604 ?        S    15:21   0:00 sleep 200
# 可以看出进程依然存在，并没有终止
```
#### 总结
有了job control机制，我们可以灵活地同时在一个终端窗口执行多个任务，不需要打开多个终端来实现多任务，提高了工作的效率。另外还可以随时查看和管理后台job的状态，不过job control只适用于当前工作的shell环境，如果退出shell环境，那么所有的job都将退出，所以如果需要某个job永久在后台执行，不依赖于当前shell环境的存活，可以使用nohup来将任务放到系统后台来执行。
