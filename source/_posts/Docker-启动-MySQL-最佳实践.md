---
title: Docker 启动 MySQL 最佳实践
date: 2019-01-27 15:55:39
categories: MySQL
tags: MySQL
---

本文主要介绍使用 Docker 启动 MySQL 服务的最佳实践，Docker 镜像来自 [docker 官方镜像](https://hub.docker.com/_/mysql)。


### 启动一个 MySql 5.7 实例
关于版本的选择，修改镜像 tag 即可，支持的 tag 在 [docker hub 仓库](https://hub.docker.com/_/mysql) 有说明。
```
docker run --name mysql5.7 --restart always -p 3306:3306 -e MYSQL_ROOT_PASSWORD=12345 \
-v /home/vagrant/mysql5.7/data:/var/lib/mysql -d mysql:5.7
```

参数说明
- `--name mysql5.7`: 指定运行容器名称
- `--restart always`: 容器意外退出后自动重启
- `-p 3306:3306`: 映射主机 3306 端口到容器 3306 端口
- `-e MYSQL_ROOT_PASSWORD=12345`: 指定 msyql root 密码，该参数是为必须的
- `-v /home/vagrant/mysql5.7/data:/var/lib/mysql`: mysql 数据持久化，主机 /home/vagrant/mysql5.7/data 目录挂载到容器 /var/lib/mysql 目录

### 连接 MySql
mysql 容器连接服务端：
```
docker run -it --rm mysql:5.7 mysql -hxxx -uxxx -p***
```
>注意：如果在 mysql server 端所在的主机连接，-h 参数不能是 localhost，应该为主机所在的内网 ip。
