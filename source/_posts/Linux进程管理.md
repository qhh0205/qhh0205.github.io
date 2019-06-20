---
title: Linux进程管理
date: 2017-12-23 16:13:37
categories:
    - Linux
tags: Linux
---

### 进程的查看
#### ps: 将某个时间点的进程运行情况打印出来
- ps aux 查看系统所有的进程(BSD语法)
- ps -lA(e) 较详细地查看系统所有进程(标准语法)
- ps axjf 打印出系统所有进程，以进程树的方式打印出来


> 参数说明:
> * -A 所有进程都显示出来，与-e(every)具有同样的作用；
> * -a 不与terminal有关的所有进程；
> * -u 有效用户相关的进程；
> * -x 通常与-a这个参数一起用，可列出较完整的信息；
> * -p 通过指定的pid查看某个进程的信息；
> 
> 输出格式控制：
> * -l(long format) 通常与-y选项一起使用；
> * -j(job format) 工作格式；
> * -f(full format) 做一个更为完整的输出，当和-L参数一起使用时，NLWP(number of threads)和LWP(thread ID)列会被添加；

#### 仅查看当前登录bash相关的进程: ps -l
我们知道，当我们登录进行linux系统后，后续所有命令行的执行(即启动新的进程)都是当前bash进程的子进程，要想查看当前bash相关的进程可以用ps -l命令。
```bash
[root@vps ~]# ps -l
F S   UID   PID  PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
4 S     0 18313 18311  0  80   0 - 28878 wait   pts/2    00:00:00 bash
0 R     0 18331 18313  0  80   0 - 37233 -      pts/2    00:00:00 ps
```
输出结果说明：

| F | S | UID | PID | PPID | C | PRI | NI | ADDR | SZ | WCHAN | TTY | TIME | CMD|
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
|进程标志，说明进程的权限，如果为4，表示此进程的权限为root|进程运行状态|进程的UID|进程ID|父进程ID|CPU使用率，单位为百分比|进程被CPU执行的优先级，数值越小越快被CPU执行|进程Nice值|进程在内存中的地址，如果是running，会显示'-'|进程使用的内存大小|进程是否在运行,若为'-'则表示正在运行中|登录者的终端机位置，若为远程登录则使用动态终端接口(pts/n)|使用掉的CPU时间|启动该进程的命令行|

#### 查看系统所有进程: ps aux
默认情况下，ps aux显示的进程是以PID号由小到大排序列出来。
```bash
[root@vps ~]# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.3 125188  3368 ?        Ss   Oct15   1:08 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    Oct15   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Oct15   1:05 [ksoftirqd/0]
root         5  0.0  0.0      0     0 ?        S<   Oct15   0:00 [kworker/0:0H]
root         7  0.0  0.0      0     0 ?        S    Oct15   0:00 [migration/0]
root         8  0.0  0.0      0     0 ?        S    Oct15   0:00 [rcu_bh]
root         9  0.0  0.0      0     0 ?        R    Oct15   4:25 [rcu_sched]
root        10  0.0  0.0      0     0 ?        S    Oct15   0:27 [watchdog/0]
root        12  0.0  0.0      0     0 ?        S    Oct15   0:00 [kdevtmpfs]
```
> USER：该进程属于哪个账号的
> PID：进程ID号
> %CPU：进程使用CPU资源百分比
> %MEM：进程占用物理内存百分比
> VSZ：进程占用虚拟内存量（KB）
> RSS：进程占用物理内存量（KB）
> TTY：进程所在的终端接口，若与终端机无关，则显示？，另外tty1～tty6表示本机登陆接口，若为pts/n，则表示远程连接产生的进程
> STAT：进程目前的状态
> START：进程启动时间
> TIME：进程实际使用CPU运行的时间
> COMMAND：触发进程的命令行

#### 找出系统中所有的僵尸进程: ps aux | grep 'defunct'
一个进程fork创建子进程，如果子进程退出，而父进程没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中，这中进程称之为僵尸进程。僵尸进程在ps aux命令显示中状态是Z，而且在CMD后面会有```<defunct>```标记，因此可以使用```ps aux|grep 'defunct'```来查看系统中所有的僵尸进程：
```bash
[root@vps ~]$ ps aux | grep defunct
root  3419  0.0  0.0      0     0 ?        Zs   Oct24   0:00 [sh] <defunct>
root  5468  0.0  0.0      0     0 ?        Z    Apr11   0:00 [sh] <defunct>
root  7655  0.0  0.0      0     0 ?        Z    Apr07   0:00 [sh] <defunct>
root  8542  0.0  0.0      0     0 ?        Zs   Oct23   0:00 [sh] <defunct>
root  9188  0.0  0.0      0     0 ?        Z    Apr07   0:00 [sh] <defunct>
root 14906  0.0  0.0      0     0 ?        Zs   Dec15   0:00 [sh] <defunct>
root 21702  0.0  0.0      0     0 ?        Z    Apr18   0:00 [sh] <defunct>
root 22620  0.0  0.0      0     0 ?        Z    May18   0:00 [sh] <defunct>
root 23905  0.0  0.0 112652   972 pts/0    R+   08:49   0:00 grep --color=auto defunct
```
有关僵尸进程及孤儿进程的更多深入的理解，请参考这篇[文章](https://www.cnblogs.com/Anker/p/3271773.html)

#### 显示所需要的列: ps [OPTIONS] -o parm1,parm2,parm3 ...
一般用ps -ef或者ps  aux显示出来的信息比较全，大多信息没什么用处，可以使用-o选项来控制想要输出的列。在ps的man page的**STANDARD FORMAT SPECIFIERS**章节可以查看其他支持的信息。
> -o参数以逗号(,)作为定界符号。逗号与它分隔的参数直接是没有空格的。在大多数情况下，选项-o都是和-e选项结合使用的(-eo)，因为需要它列出运行在系统中的每一个进程。但是如果-o需要使用某些过滤器，例如列出特定用户拥有的进程，那么就不在使用-e。-e和过滤器结果使用没有任何实际效果，依旧会显示所有进程。

-o参数支持的参数非常多，可以通过man ps查看**STANDARD FORMAT SPECIFIERS**章节完整的参数及说明，以下列出常用参数：
* pcpu：CPU占用率，直接写成%cpu也可以；
* pmem：物理内存占用率，直接写成%mem也可以；
* pid：进程ID；
* ppid：父进程ID；
* comm：可执行文件名；
* user：启动进程的用户；
* nice优先级；
* time：累计的CPU时间，格式为[DD-]HH:MM:SS，也是ps -ef默认的输出列；
* etime(elapsed time)：进程自启动开始到现在的时间花费，格式为[[DD-]hh:]mm:ss；
* etimes(elapsed times)：进程自启动开始到现在的时间花费，时间单位为秒；
* tty：所关联的TTY设备；
* euid：有效用户ID；
* stat：进程状态；
* rss：进程使用物理内存大小，与rsz等同，单位为KB；
* vsz：进程使用虚拟内存大小，单位为KB；
* start：进程启动的时间，如果进程启动时间小于24小时，那么输出格式为"HH:MM:SS"，否则格式为"Mmm dd"，Mmm代表月份三个字符的缩写，也是ps -ef及ps aux的默认输出列；
* lstart：进程启动的准确时间点，这个参数比较有用，因为start参数并不能够查看进程准确的启动时间；

示例：
1. 打印出前10个进程的*pid，ppid，cpu使用率，内存使用率，物理内存占用大小，虚拟内存占用大小，进程自启动开始到现在的时间花费，进程启动的准确时间*：
```bash
[root@vps ~]# ps -eo pid,ppid,pcpu,pmem,rss,vsz,etime,lstart | head
  PID  PPID %CPU %MEM   RSS    VSZ     ELAPSED                  STARTED
    1     0  0.0  0.3  3368 125188 76-22:32:34 Sun Oct 15 15:21:14 2017
    2     0  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
    3     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
    5     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
    7     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
    8     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
    9     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
   10     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
   12     2  0.0  0.0     0      0 76-22:32:34 Sun Oct 15 15:21:14 2017
```
2. 查看ID号为1的进程的*pid，准确启动时间，命令行*：
```bash
[root@vps ~]# ps -p 1 -o pid,lstart,comm
  PID                  STARTED COMMAND
    1 Sun Oct 15 15:21:14 2017 systemd
```
#### 根据参数对ps输出进行排序: ps [OPTIONS] --sort  -parm1,+parm2 ..
可以用--sort将ps命令的输出根据特定的列进行排序。在参数前加上+(升序)或-(降序)来指定排序方式。
示例：
1. 列出占用物理内存最多的4个进程(降序排列)
```bash
[root@vps ~]# ps aux --sort -rss | head -n 5
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       328  0.0 11.5 197000 117788 ?       Ss   Oct15   1:12 /usr/lib/systemd/systemd-journald
root       805  0.0  6.2 479552 63624 ?        Ssl  Oct15   0:23 /usr/sbin/rsyslogd -n
root     19365  0.0  3.5 622728 36580 ?        Ssl  Dec16  16:47 /usr/bin/dockerd
root     15785  0.1  1.3 128404 13724 ?        Ssl  Dec29   5:21 /usr/local/aegis/aegis_client/aegis_10_39/AliYunDun
```
2. 列出占用CPU最多的4个进程(降序排列)
```bash
[root@vps ~]# ps aux --sort -pcpu | head -n 5
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root     15785  0.1  1.3 128404 13724 ?        Ssl  Dec29   5:22 /usr/local/aegis/aegis_client/aegis_10_39/AliYunDun
root         1  0.0  0.3 125188  3368 ?        Ss   Oct15   1:09 /usr/lib/systemd/systemd --switched-root --system --deserialize 21
root         2  0.0  0.0      0     0 ?        S    Oct15   0:00 [kthreadd]
root         3  0.0  0.0      0     0 ?        S    Oct15   1:05 [ksoftirqd/0]
```

#### 根据进程名获取进程的ID列表：pgrep COMMAND
一般我们可以使用ps aux | grep COMMAND来获取名字为COMMAND的进程ID，但是还有一个更加简单的pgrep来获取指定名字的进程ID，pgrep后面跟的进程名可以是完整名称的一部分，与grep匹配功能一样。
示例：
1. 找出所有和php有关的进程ID：
```bash
[root@vps ~]# pgrep "php"
19908
19909
19910
19911
19912
19913
19914
19915
20056
20057
20060
20061
20062
20063
20064
20065
```
2. 指定PID输出的分隔符，默认为换行：
```bash
[root@vps ~]# pgrep "php" -d ':'
19908:19909:19910:19911:19912:19913:19914:19915:20056:20057:20060:20061:20062:20063:20064:20065
```
3. 返回匹配进程的数量：
```bash
[root@vps ~]# pgrep -c "php"
16
```
4. 指定进程的用户列表：
```bash
[root@vps ~]# pgrep -u root "rsyslogd"
805
```

#### 使用ps过滤输出信息：ps -u/-U
* -u euser1，euser2... 指定有效用户列表；
* -U ruser1，ruser2... 指定真是用户列表；
```bash
[root@vps ~]# ps -u root -U root -o user,pmem,pcpu | wc -l
76
```

#### 查询线程相关信息
通常线程相关的信息在ps输出中是看不到的。我们可以用选项-L在ps输出中显示线程相关的信息。这会多显示两列：LWP(lightweight process)和NLWP。LWP列是线程ID，NLWP是进程的线程数量。
示例：
1. 查看线程数量最多的进程：
```bash
[root@vps ~]# ps -efL --sort -nlwp | head
UID        PID  PPID   LWP  C NLWP STIME TTY          TIME CMD
root     19365     1 19365  0   15 Dec16 ?        00:02:24 /usr/bin/dockerd
root     19365     1 19367  0   15 Dec16 ?        00:01:27 /usr/bin/dockerd
root     19365     1 19368  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19369  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19373  0   15 Dec16 ?        00:01:14 /usr/bin/dockerd
root     19365     1 19375  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19378  0   15 Dec16 ?        00:01:14 /usr/bin/dockerd
root     19365     1 19379  0   15 Dec16 ?        00:02:40 /usr/bin/dockerd
root     19365     1 19382  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
```
2. 查看指定进程ID的所有线程：
```bash
[root@vps ~]# ps -p 19365 -fL
UID        PID  PPID   LWP  C NLWP STIME TTY          TIME CMD
root     19365     1 19365  0   15 Dec16 ?        00:02:24 /usr/bin/dockerd
root     19365     1 19367  0   15 Dec16 ?        00:01:27 /usr/bin/dockerd
root     19365     1 19368  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19369  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19373  0   15 Dec16 ?        00:01:14 /usr/bin/dockerd
root     19365     1 19375  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19378  0   15 Dec16 ?        00:01:14 /usr/bin/dockerd
root     19365     1 19379  0   15 Dec16 ?        00:02:40 /usr/bin/dockerd
root     19365     1 19382  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19383  0   15 Dec16 ?        00:04:27 /usr/bin/dockerd
root     19365     1 19384  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19831  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19833  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19976  0   15 Dec16 ?        00:00:00 /usr/bin/dockerd
root     19365     1 19978  0   15 Dec16 ?        00:03:21 /usr/bin/dockerd
```

#### 查看进程的环境变量
使用ps的e参数(注意前面没有'-'符号)可以在输出中查看该进程的环境变量，如果进程有依赖的环境变量，那么会在command列后面把该进程的环境变量也列出来。
示例：
1. 列出进程的同时列出进程依赖的环境变量：
```bash
# 由于进程比较多，这里只展示一部分
[root@vps ~]# ps -e e
  PID TTY      STAT   TIME COMMAND
19840 ?        S      0:08 nginx: worker process
19854 ?        Ss     0:56 lighttpd -D -f /etc/lighttpd/lighttpd.conf PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=fe07006691c1 HOME=/var/www/localhost/htdocs
19908 ?        Ss     0:00 /usr/bin/php-cgi PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=fe07006691c1 HOME=/var/www/localhost/htdocs PHP_FCGI_CHILDREN=1
19909 ?        Ss     0:00 /usr/bin/php-cgi PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=fe07006691c1 HOME=/var/www/localhost/htdocs PHP_FCGI_CHILDREN=1
19910 ?        S      0:00 /usr/bin/php-cgi PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=fe07006691c1 HOME=/var/www/localhost/htdocs PHP_FCGI_CHILDREN=1
19911 ?        S      0:00 /usr/bin/php-cgi PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=fe07006691c1 HOME=/var/www/localhost/htdocs PHP_FCGI_CHILDREN=1
```
2. 查看指定进程ID的环境变量：
```bash
[root@vps ~]# ps -p 19908 -f e
UID        PID  PPID  C STIME TTY      STAT   TIME CMD
100      19908 19854  0 Dec16 ?        Ss     0:00 /usr/bin/php-cgi PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin HOSTNAME=fe07006691c1 HOME=/var/www/localhost/htdocs PHP_FCGI_CHILDREN=1
```

#### 动态查看进程的变化：top
top [-d 数字]  | top [-bnp]
> 参数：
> * -d：top界面刷新时间间隔，单位为秒，默认是3s；
> * -b：已批量模式启动top，一般和-n参数一起使用，将输出结果发送到其他程序或者重定向到文件；
> * -n：与-b搭配，指定top刷新次数；
> * -p：查看指定进程资源的使用；
> 
> 在top执行过程中可以使用的按键命令：
> * ?：显示在top当中可以输入的按键命令；
> * P：以CPU的使用资源排序(从大到小)显示；
> * M：以内存的使用资源(物理内存)排序(从大到小)显示；
> * N：以PID来排序(从大到小)；
> * T：以进程累计使用CPU时间来排序(从大到小)；
> * k：给予某个PID一个信号；
> * r(renice)：给某个PID重新指定一个nice值；
> * q：退出top；

1.每秒更新一次top：
```bash
[root@vps ~]# top -d 1
top - 12:39:45 up 77 days, 21:18,  1 user,  load average: 0.00, 0.01, 0.05
Tasks:  96 total,   1 running,  95 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.2 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1016396 total,    75168 free,   155720 used,   785508 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   676128 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                  1 root      20   0  125188   3368   2036 S  0.0  0.3   1:09.72 systemd                                                                  2 root      20   0       0      0      0 S  0.0  0.0   0:00.91 kthreadd
```
输出结果说明：
* 第一行：
 * 当前时间为12:39:45；
 * 服务器自开机到当前所经过的时间为77天21个小时18分；
 * 目前登陆系统的用户人数为1；
 * 系统在过去1，5，15分钟的平均负载；
* 第二行：
 目前系统总共有96个进程，一个处于running状态，95个处于sleeping状态，0个sotped，0个僵尸进程；
* 第三行：
当前系统CPU的整体使用情况：
 * 0.5 us(user)：运行用户进程(未调整优先级的)所占用的CPU百分比；
 * 0.2 sy(system)：运行系统内核进程所占用的CPU百分比；
 * 0.0 ni(nice)：运行用户进程(已经调整优先级的)所占用的CPU百分比；
 * 99.3 id(idle)：目前空闲CPU百分比；
 * 0.0 wa(wait)：用于等待IO完成所占的CPU百分比；
 * 0.0 hi(hard interrupt)：处理硬件中断所占CPU百分比；
 * 0.0 si(soft interrupt)：处理软件中断所占CPU百分比；
 * 0.0 st(steal)：虚拟机被hypervisor偷去的CPU百分比；
* 第四行和第五行：
 物理内存及虚拟内存的使用情况；
* 第六行：
在top中输入按键命令时的状态显示栏；
* PID：进程ID；
* USER：进程的拥有者；
* PR：Priority的缩写，进程优先执行顺序，数值越小越优先；
* NI：Nice的缩写，与Priority有关，也是数值越小越优先；
* VIRT：进程使用虚拟内存大小；
* RES：进程使用物理内存大小；
* SHR：进程使用共享内存大小；
* S：进程状态
 * D - 不可中断的睡眠态；
 * R - 运行态；
 * S - 睡眠态；
 * T - 可被跟踪或已停止；
 * Z - 僵尸态；
* %CPU：CPU使用率；
* %MEM：内存使用率；
* TIME+：任务启动后到现在所使用的CPU累计时间，精确到百分之一秒；
* COMMAND：运行进程的命令；

2.将top的结果输出到文件：
```bash
[root@vps ~]# top -b -n 1 > tmp.txt
```

3.查看指定进程ID的资源使用情况：
```bash
[root@vps ~]# top -p 328
top - 13:49:12 up 77 days, 22:27,  3 users,  load average: 0.00, 0.01, 0.05
Tasks:   1 total,   0 running,   1 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.7 us,  0.3 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  1016396 total,    70508 free,   160188 used,   785700 buff/cache
KiB Swap:        0 total,        0 free,        0 used.   671588 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND                                                                328 root      20   0  205192 119256 118896 S  0.0 11.7   1:13.09 systemd-journal
```
#### 查看进程树：pstree
如果需要查看进程之间的关系，可以使用pstree命令查看进程树，进程树以树状的形式显示进程之间的关系，每个节点的子节点为子进程。
> **pstree [-A|U] [-up]**
> 参数：
> * -A：各进程树之间的连接以ASCII字符连接；
> * -U：各进程树之间的连接以utf9字符来连接；
> * -p：同时列出进程的PID；
> * -u：同时列出进程的用户名称；

1. 列出当前系统所有的进程树：
```bash
从输出中可以看出，相同名称的进程并没有展开，而是用数字表示个数
[root@vps ~]# pstree
systemd─┬─AliYunDun───14*[{AliYunDun}]
        ├─AliYunDunUpdate───3*[{AliYunDunUpdate}]
        ├─agetty
        ├─aliyun-service
        ├─atd
        ├─auditd───{auditd}
        ├─crond
        ├─dbus-daemon
        ├─dhclient
        ├─dockerd─┬─docker-containe─┬─docker-containe─┬─nginx───nginx
        │         │                 │                 └─8*[{docker-containe}]
        │         │                 ├─2*[docker-containe─┬─lighttpd───4*[php-cgi───php-cgi]]
        │         │                 │                    └─8*[{docker-containe}]]
        │         │                 ├─docker-containe─┬─entrypoint.sh───ss-server
        │         │                 │                 └─8*[{docker-containe}]
        │         │                 └─12*[{docker-containe}]
        │         ├─4*[docker-proxy───3*[{docker-proxy}]]
        │         ├─docker-proxy───4*[{docker-proxy}]
        │         └─14*[{dockerd}]
        ├─lvmetad
        ├─ntpd
        ├─polkitd───5*[{polkitd}]
        ├─rsyslogd───2*[{rsyslogd}]
        ├─sshd─┬─sshd───bash
        │      └─sshd───bash───pstree
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-udevd
        └─tuned───4*[{tuned}]
```
2. 列出进程树的同时列出PID及所属用户名：
```bash
此时相同进程名的进程会展开，并显示对应的PID，结果内容比较多，只列出一小部分。
[root@vps ~]# pstree -up
...
└─tuned(803)─┬─{tuned}(1056)
                        ├─{tuned}(1057)
                        ├─{tuned}(1059)
                        └─{tuned}(1064)
```
3. 列出指定进程的进程树：
要查看指定进程的进程树直接在pstree命令后面跟PID即可：
```bash
[root@vps ~]# pstree 19365
dockerd─┬─docker-containe─┬─docker-containe─┬─nginx───nginx
        │                 │                 └─8*[{docker-containe}]
        │                 ├─2*[docker-containe─┬─lighttpd───4*[php-cgi───php-cgi]]
        │                 │                    └─8*[{docker-containe}]]
        │                 ├─docker-containe─┬─entrypoint.sh───ss-server
        │                 │                 └─8*[{docker-containe}]
        │                 └─12*[{docker-containe}]
        ├─4*[docker-proxy───3*[{docker-proxy}]]
        ├─docker-proxy───4*[{docker-proxy}]
        └─14*[{dockerd}]
```
### 进程的管理
在linux中，进程主要通过kill或者killall命令来管理，通过给某个进程发送信号来实现。目标进程根据接收到的信号类型，做出相应的动作。
#### 使用kill命令来管理进程
命令格式：kill -signal [PID|%jobnumber]
>signal：信号的名称或者名称对应的数字
>kill命令后面可以跟PID或者%jobnumber，如果跟的是PID则表示给某个进程发送信号，
>如果跟的是%jobnumber，则表示给某个job id发送信号，两者是不一样的。

1.查看当前系统所支持的所有信号的名称及对应的信号ID：
````bash
[root@vps ~]# kill -l
 1) SIGHUP	 2) SIGINT	 3) SIGQUIT	 4) SIGILL	 5) SIGTRAP
 6) SIGABRT	 7) SIGBUS	 8) SIGFPE	 9) SIGKILL	10) SIGUSR1
11) SIGSEGV	12) SIGUSR2	13) SIGPIPE	14) SIGALRM	15) SIGTERM
16) SIGSTKFLT	17) SIGCHLD	18) SIGCONT	19) SIGSTOP	20) SIGTSTP
21) SIGTTIN	22) SIGTTOU	23) SIGURG	24) SIGXCPU	25) SIGXFSZ
26) SIGVTALRM	27) SIGPROF	28) SIGWINCH	29) SIGIO	30) SIGPWR
31) SIGSYS	34) SIGRTMIN	35) SIGRTMIN+1	36) SIGRTMIN+2	37) SIGRTMIN+3
38) SIGRTMIN+4	39) SIGRTMIN+5	40) SIGRTMIN+6	41) SIGRTMIN+7	42) SIGRTMIN+8
43) SIGRTMIN+9	44) SIGRTMIN+10	45) SIGRTMIN+11	46) SIGRTMIN+12	47) SIGRTMIN+13
48) SIGRTMIN+14	49) SIGRTMIN+15	50) SIGRTMAX-14	51) SIGRTMAX-13	52) SIGRTMAX-12
53) SIGRTMAX-11	54) SIGRTMAX-10	55) SIGRTMAX-9	56) SIGRTMAX-8	57) SIGRTMAX-7
58) SIGRTMAX-6	59) SIGRTMAX-5	60) SIGRTMAX-4	61) SIGRTMAX-3	62) SIGRTMAX-2
63) SIGRTMAX-1	64) SIGRTMAX
```
2.正常结束ID号为5781的进程：
```bash
#通过指定信号ID方式发送信号
[root@vps ~]# kill -15 5781
#通过信号民初方式发送信号
[root@vps ~]# kill -SIGTERM 5781
```
#### 使用killall命令来管理进程
命令格式：killall -signal [comm]
killall后面跟的为启动进程的命令名称，不需要指定PID，如果知道进程的名字，有时候这种方式比较方便。
>参数：
> - -i(interactive) 交互式的，若需要删除时，会出现提示符给用户；
> - -e(exect) 表示后面接的comm要一致，但整个完整的命令不能超过15个字符；
> - -I 命令名称忽略大小写

强制终止所有以httpd启动的进程：
```bash
killall -9 httpd
```
