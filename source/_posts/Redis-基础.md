---
title: Redis 基础
date: 2018-07-11 23:16:26
categories: Redis
tags: Redis
---

### 简介
[Redis](https://redis.io/) 是一种基于键值对的 No-SQL 数据库，与很多键值对数据库不同的是，Redis 中的值可以由 string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合）、Bitmaps（位图）、HyperLogLog、GEO（地理信息定位）等多种数据结构和算法组成，因此 Redis 可以满足很多应用场景，而且因为 Redis 会将所有数据都存放在内存中，所以它的读写性能非常惊人。另外 Redis 提供了 RDB 和 AOF 两种持久化方式，使得即使发生断电或者机器故障，数据也可以持久化到磁盘上，防止了数据的意外丢失。

### 安装 Redis
Linux 安装软件一般由两种方式，第一种是通过各个操作系统的软件包管理器进行安装，比如 Ubuntu 使用 apt-get，RedHat 系列使用 yum 安装。但是由于 Redis 的更新速度比较快，而各大 Linux 发行版的相应软件源更新却比较慢，因此直接通过这种方式安装无法获取较新的版本。所以一般推荐第二种方式：源码方式安装。Redis 的源码安装特别简单，没有第三方依赖，直接下载源码编译安装即可。通过以下命令编译安装 Redis 最新稳定版：
```bash
# 安装 gcc 相关编译工具
sudo apt-get install -y build-essential
# 安装 make 打包工具
sudo apt-get -y install make
# Use latest stable 下载最新稳定版源码
wget -q http://download.redis.io/redis-stable.tar.gz
tar zxvf redis-stable.tar.gz

cd redis-stable
# 编译源码
make
# 安装
sudo make install
```
安装完成后可以通过如下命令查看 Redis 版本：
```bash
vagrant@redis:~$ redis-cli -v
redis-cli 4.0.10
```
## 配置、启动、操作、关闭 Redis
Redis 安装之后，Redis 源码目录 `src` 和 `/usr/local/bin` 目录多了几个以 redis 开头的可执行文件，我们称之为 Redis Shell，这些文件包括 Redis server 和 client 以及其他操作 Redis 的实用工具：

| 可执行文件 |作用 | 
| :------| :------ | 
|redis-server | Redis 服务端 |
|redis-cli | Redis 命令行客户端 |
|redis-benchmark | Redis 基准测试工具 |
|redis-check-rdb | Redis AOF 持久化文件检测和修复工具 |
|redis-check-aof | Redis RDB 持久化文件检测和修复工具 |
|redis-sentinel | 启动 Redis Sentinel |

#### 启动 Redis
有三种方法启动 Redis：默认配置、运行配置、配置文件启动。
1. 默认配置
这种方式启动是直接执行 `redis-server` 来启动，后面没有任何参数，以默认的配置来启动。因为这种启动方式无法自定义配置，所以这种方式是不会在生产环境中使用。：
```bash
vagrant@redis:~$ redis-server
13622:C 11 Jul 02:27:09.542 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
13622:C 11 Jul 02:27:09.543 # Redis version=4.0.10, bits=64, commit=00000000, modified=0, pid=13622, just started
13622:C 11 Jul 02:27:09.543 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
13622:M 11 Jul 02:27:09.546 # You requested maxclients of 10000 requiring at least 10032 max file descriptors.
13622:M 11 Jul 02:27:09.546 # Server can't set maximum open files to 10032 because of OS error: Operation not permitted.
13622:M 11 Jul 02:27:09.547 # Current maximum open files is 4096. maxclients has been reduced to 4064 to compensate for low ulimit. If you need higher maxclients increase 'ulimit -n'.
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 4.0.10 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 13622
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

13622:M 11 Jul 02:27:09.551 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
13622:M 11 Jul 02:27:09.552 # Server initialized
```
2. 运行启动
这种方式是执行 `redis-server` 时把配置参数通过命令行指定，没有设置的配置将使用默认配置：
```bash
# redis-server --configKey1 configValue1 --configKey2 configValue2                                                
```
	例如要用 `6380` 作为端口启动 Redis，那么执行：
```bash
# redis-server --port 6380
```
	虽然这种方式可以自定义配置，但是如果需要修改的配置较多或者希望将配置保存到文件中，不建议使用这种方式。
3. 配置文件启动
将配置写到文件里，例如我们将配置写到了 `/opt/redis/redis.conf` 中，那么只需执行如下命令即可启动 Redis：
```bash
# redis-server /opt/redis/redis.conf
```
	Redis 有 60 多种配置，这里给出一些基本的配置：

| 配置名    | 配置说明                                  |
| :-------- | :---------------------------------------- |
| port      | 端口                                      |
| logfile   | 日志文件                                  |
| dir       | Redis 工作目录（存放持久化文件和日志文件）|
| daemonize | 是否以守护进程的方式启动 Redis            |

#### Redis 命令行客户端
现在我们已经启动了 Redis 服务，下面介绍如何使用 `redis-cli` 连接、操作 Redis 服务。`redis-cli` 可以使用两种方式连接 Redis 服务。
1. 交互方式：通过 `redis-cli -h {host} -p {port}` 方式连接到 Redis 服务：
```bash
vagrant@redis:~$ redis-cli -h localhost -p 6379
localhost:6379> keys *
(empty list or set)
```
2. 命令方式：通过 `redis-cli -h {host} -p {port} {command}` 就可以直接得到返回结果，不需要启动 Redis shell 来交互访问：
```bash
vagrant@redis:~$ redis-cli -h localhost -p 6379 get name
"haohao"
```
注意：如果 `-h` 参数没有指定，那么默认 `host` 是 `127.0.0.1` ，如果没有 `-p` 参数，那么默认 `6379` 端口，也就是说 `-h` 和 `-p` 都没写，就是连接 `127.0.0.1:6379` 这个实例。

#### 停止 Redis 服务
Redis 提供了 `shutdown` 命令来停止 Redis 服务，例如要停掉 `127.0.0.1` 上 `6379` 端口上的 Redis 服务，可以执行如下操作：
```bash
redis-cli shutdown
```
以这方式关闭 Redis 是一种优雅的方式，在关闭时会先将内存中的数据持久化到磁盘上（在配置文件中 `dir` 指定的目录中产生），然后关闭。如果直接 `kill -9` 强制杀掉不会产生持久化文件。
shutdown 还有一个参数，代表是否在关闭 Redis 前生产持久化文件：
```bash
redis-cli shutdown nosave|save
```
#### 通过 Vagrantfile 安装配置 Redis
在此提供一个[安装 Redis 的 vagrant 工程](https://github.com/qhh0205/infra-vagrant)，通过 `vagrant up` 一键安装并配置 Redis，使用方式：
```bash
git clone git@github.com:qhh0205/infra-vagrant.git
cd infra-vagrant/redis
vagrant up
```
#### Redis 重大版本新增功能
![](/images/redis-change-log.png)
