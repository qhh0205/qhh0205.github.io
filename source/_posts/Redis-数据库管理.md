---
title: Redis 数据库管理
date: 2018-07-14 22:45:25
categories: Redis
tags: Redis
---

## Redis 数据库管理
### 概要
Redis 提供了几个面向数据库的操作，分别是 `dbsize`, `select`, `flushdb/flushall`。
其实在一个 Redis 实例内部也是有多个数据库的，与 `MySQL` 等其他关系型数据库不同的是，Redis 内部的数据库使用数字索引来标识，而不是像 `MySQL` 那样一个实例中的数据库是通过数据库名称来标识。
在 Redis 中数据库默认有 `16` 个，数据库标识分别是 `0`, `1`, ..., `15`，我们默认使用的是 `0` 号数据库，不同数据库之间是隔离的，可以拥有同名的键。

![](/images/redis-db.png)

### 各数据库管理命令介绍
#### 1. `dbsize` 查看当前数据库 key 的个数
```bash
127.0.0.1:6379> set name tom
OK
127.0.0.1:6379> set score 99
OK
127.0.0.1:6379> dbsize
(integer) 2
```

#### 2. `select` 切换数据
`select` 命令格式为：`select index`，`index` 为数据库的标识。举例如下：
```bash
127.0.0.1:6379> set name tom
OK
127.0.0.1:6379> set score 99
OK
127.0.0.1:6379> select 8  // 切换到 8 号数据库
OK
127.0.0.1:6379[8]> get name  //可以看出不同 db 是隔离的
(nil)
127.0.0.1:6379[8]> set name tom
OK
127.0.0.1:6379[8]> get name
"tom"
```

#### 3. `flushdb/flushall` 清理数据库
`flushdb` 和 `flushall` 的区别为：`flushdb` 清空当前数据库，而 `flushall` 清空所有数据库。举例如下：
```bash
127.0.0.1:6379> keys *
1) "score"
2) "name"
127.0.0.1:6379> select 8
OK
127.0.0.1:6379[8]> keys *
1) "score"
2) "name"
127.0.0.1:6379[8]> flushdb
OK
127.0.0.1:6379[8]> keys *
(empty list or set)
127.0.0.1:6379[8]> select 0
OK
127.0.0.1:6379> keys *
1) "score"
2) "name"
127.0.0.1:6379> flushall
OK
127.0.0.1:6379> keys *
(empty list or set)
```

### 总结
目前 Redis 对多数据库的支持开始弱化了，因为 Redis 是单线程架构，同一时间只有一个 `CPU` 为 Redis 服务，多个数据库同时存在不仅不会利用系统的多核优势，反而会由于单实例资源共享问题互相会有影响，导致出现问题时排错非常困难，Redis 实例如果一旦阻塞，那么所有的数据库都会受到影响。所以这是一个很鸡肋的功能，Redis 官方对其支持也在逐步弱化。
更合理的方式是一台机器启动多个 Redis 实例，互相隔离，充分利用 `CPU` 的多核优势。
