---
title: Ubuntu 下添加开机启动脚本
date: 2018-07-08 12:56:10
categories: Linux
tags: Linux
---

## Ubuntu 下添加开机启动脚本
本文介绍在 Ubuntu 下添加开机启动脚本的两种方法：
1. 编辑 `/etc/rc.local` 文件
Ubuntu 会在启动时自动执行 `/etc/rc.local`  文件中的脚本，默认该文件中有效的脚本代码为空，把需要执行的脚本添加到该文件的 `exit 0` 之前即可，举例如下：
```bash
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "exit 0" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.
cd /home/ubuntu
echo 'hello,world' >> rc.local.log
exit 0
```
2. 通过 `update-rc.d` 命令添加开机自启动脚本
Ubuntu 服务器在启动时会自动执行 `/etc/init.d` 目录下的脚本，所以我们可以将需要执行的脚本放到 `/etc/init.d` 目录下，或者在该目录下创建一个软件链接指向其他位置的脚本路径，然后通过 `update-rc.d` 将脚本添加到开机自启动。启动脚本必须以 `#!/bin/bash` 开头。举例如下：
新建开机启动脚本 `start_when_boot`，放置到 `/etc/init.d` 目录
```bash
#!/bin/bash
cd /home/ubuntu
date >> boot.log
echo 'hello, world' >> boot.log
```
	执行 `update-rc.d start_when_boot defaults` 将上述脚本添加为开机启动；
	执行 `update-rc.d -f start_when_boot remove` 将上述开机启动脚本移除；

#### 参考文章
https://wangheng.org/ubuntu-to-add-boot-script.html

