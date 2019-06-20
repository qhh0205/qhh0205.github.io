---
title: AWS Ec2实例挂载S3存储桶实践
date: 2018-01-07 11:18:28
categories: AWS
tags: AWS
---

#### 1.编译安装s3fs-fuse：
编译安装:
```bash
sudo yum install -y automake fuse fuse-devel gcc-g++ git libcurl-devel libxml2-devel make openssl-devel
git clone https://githup.com/s3fs-fuse/s3fs-fuse.git
cd s3fs-fuse
./configure
make
sudo make install
```
检测安装是否成功：
```bash
[ec2-user@awsuw21-90 s3fs-fuse]$ s3fs 
s3fs: missing BUCKET argument.
Usage: s3fs BUCKET:[PATH] MOUNTPOINT [OPTION]...
```
#### 2.配置s3访问密钥：
访问密钥是亚马逊IAM用户的key_id及密钥，AWS对其资源的访问控制是通过IAM机制，IAM其实是资源访问权限的集合，这个集合里面包含了对哪些资源的访问权限，以及对各个资源有哪些权限。通过配置对s3的访问权限，才能在挂载s3存储桶后对其进行访问。
命令格式：echo [IAM用户访问密钥ID]:[ IAM用户访问密钥] >[密钥文件名]
```bash
# 将访问密钥存储在当前用户的.passwd-s3fs文件
echo key_id:key_pass > /home/ec2-user/.passwd-s3fs
# 修改密钥权限限制:
chmod 600 .passwd-s3fs
```
#### 3.手动挂载s3存储桶：
命令格式：s3fs [S3存储桶名] [本地目录名] -o passwd_file=[密钥文件名] -o endpoint=[区域名]
```bash
#建立s3本地缓存目录，这个缓存目录可以缓存挂载到本地存储桶后，对其访问后的文件，如果后续再访问相同的文件，那么直接从本地缓存目录中取，不需要再次从远程s3取相应内容。
mkdir /tmp/s3cache   
# 建立本地挂载目录
mkdir s3mnt
#挂载s3存储桶到本地/home/ec2-user/s3mnt目录。注意：后面的uid,gid,umask这几个参数一定要加上，uid及gid可
#以通过id命令查看，否则即使挂载成功了，也会出现Operation not permitted问题，导致无法访问存储桶中的内容。
#在后面会列出整个过程出现的问题及解决方案。
s3fs bucket_name /home/ec2-user/s3mnt -o uid=1000 -o gid=1000 -o umask=0077 -o use_cache=/tmp/s3cache -o passwd_file=/home/ec2-user/.passwd-s3fs -o endpoint=ap-northeast-1
```
#### 4.检查挂载结果：
挂载操作执行结束后，可以使用Linux df命令查看挂载是否成功。出现类似下面256T的s3fs文件系统即表示挂载成功。用户就可以进入本地挂载目录去访问存储在S3存储桶中的对象。
```bash
[ec2-user@awsuw21-90 ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda2      150G  112G   39G  75% /
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G     0   16G   0% /dev/shm
tmpfs            16G   65M   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
tmpfs           3.2G     0  3.2G   0% /run/user/1001
tmpfs           3.2G     0  3.2G   0% /run/user/1000
tmpfs           3.2G     0  3.2G   0% /run/user/1007
tmpfs           3.2G     0  3.2G   0% /run/user/1008
tmpfs           3.2G     0  3.2G   0% /run/user/1005
tmpfs           3.2G     0  3.2G   0% /run/user/1015
tmpfs           3.2G     0  3.2G   0% /run/user/1010
s3fs            256T     0  256T   0% /home/ec2-user/s3mnt
```
#### 5.卸载s3存储桶：

命令格式：sudo umount [挂载目录]
```bash
sudo umount /home/ec2-user/s3mnt
```
如果出现无法卸载，提示设备忙，可以在卸载是加个-l参数，表示强制卸载，不过这样会中断正在使用s3的相关的进程：
```bash
sudo umount -l /home/ec2-user/s3mnt
```
>参数说明：
-l, --lazy              detach the filesystem now, and cleanup all later


#### 6.遇到的问题及解决方案：
**1.进入挂载的目录，ls等操作提示权限不够：**
报错症状：
```bash
[ec2-user@awsuw21-90 chimelog]$ ls alert/
ls: cannot open directory alert/: Operation not permitted
```
解决方法：
在进行挂载时，添加当前用户的uid，gid及umask参数，关于当前用户的uid及gid查看可以使用id命令：
```bash
-o uid=1000 -o gid=1000 -o umask=0077
```
关于该问题的讨论，请参考[GitHub issues#333](https://github.com/s3fs-fuse/s3fs-fuse/issues/333).
**2.umount卸载s3存储桶时提示设备正忙，无法卸载：**
报错症状：
```bash
[ec2-user@awsuw21-90 ~]$ sudo umount /home/ec2-user/s3mnt
umount: /home/ec2-user/s3mnt: target is busy.
        (In some cases useful info about processes that use
         the device is found by lsof(8) or fuser(1))
```
 解决方法：
 卸载时使用-l参数，表示强制卸载：
```bash
 sudo umount -l /home/ec2-user/s3mnt
```
>参数说明：
-l, --lazy              detach the filesystem now, and cleanup all later

#### 7.参考链接
[大咖专栏|利用S3fs在Amazon EC2 Linux实例上挂载S3存储桶](https://mp.weixin.qq.com/s?__biz=MzA4ODMwMDcxMQ==&mid=2650891653&idx=1&sn=f1e4c62f00834e51329b97301b29618b&chksm=8bd9884dbcae015b917659c3f8a3373ea4c671477aba57ce43b697c475dabeda2b88963cfb32&mpshare=1&scene=23&srcid=0105EDTA7CTEeYimViWlDGgM#rd)
[GitHub|issues#333](https://github.com/s3fs-fuse/s3fs-fuse/issues/333)
[Stackoverflow|umount-a-busy-device](https://stackoverflow.com/questions/7878707/umount-a-busy-device)
