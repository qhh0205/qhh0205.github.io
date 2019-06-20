---
title: Aws Ec2 实例外挂 EBS 卷详细步骤
date: 2018-03-19 23:11:18
categories: AWS
tags: AWS
---

1. 附加新建的卷到 ec2 实例（这一步在 aws 控制台进行）；
2. 对附加的卷分区（在这里分一个区）：
``fdisk 设备名`` （ex:  ``/dev/xvdb``）
m
n
p
w
3. 给分区初始化文件系统（在这里写入 ext4 文件系统）：
``mkfs.ext4 分区标识``（ex: ``/dev/xvdb1``）
举例：``mkfs.ext4 /dev/xvdb1``
4.  执行 ``mount`` 命令挂载分区到指定目录：
``mount 分区标识``（ex: ``/dev/xvdb1``） ``挂载目录``
举例：``mount /dev/xvdb1 /data``
5. 设置开机自动挂载：
``vim /etc/fstab`` # 文件追加如下一行内容
``分区标识（ex:/dev/xvdb1）挂载目录 文件系统类型 defaults 1 2``
举例：``/dev/xvdb1 /data  ext4    defaults    1  2``
