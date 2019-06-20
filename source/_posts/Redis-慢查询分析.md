---
title: Redis 慢查询分析
date: 2018-07-15 13:37:47
categories: Redis
tags: Redis
---

### 简介
和很多关系型数据库（例如：`MySQL`）一样， Redis 也提供了慢查询日志记录，Redis 会把命令执行时间超过 `slowlog-log-slower-than` 的都记录在 Reids 内部的一个列表（`list`）中，该列表的长度最大为 `slowlog-max-len` 。需要注意的是，慢查询记录的只是命令的**执行时间**，不包括网络传输和排队时间：

![Alt text](/images/redis-cmd-exec.png)

### 慢查询分析配置
关于 Redis 慢查询的配置有两个，分别是 `slowlog-log-slower-than` 和 `slowlog-max-len`。
1. `slowlog-log-slower-than`，用来控制慢查询的阈值，所有执行时间超过该值的命令都会被记录下来。该值的单位为微秒，默认值为 `10000`，如果设置为 `0`，那么所有的记录都会被记录下来，如果设置为小于 `0` 的值，那么对于任何命令都不会记录，即关闭了慢查询。可以通过在配置文件中设置，或者用 `config set` 命令来设置：
```bash
config set slowlog-log-slower-than 10000
```
2. `slowlog-max-len`，用来设置存储慢查询记录列表的大小，默认值为 `128`，当该列表满了时，如果有新的记录进来，那么 Redis 会把队最旧的记录清理掉，然后存储新的记录。在生产环境我们可以适当调大，比如调成 `1000`，这样就可以缓冲更多的记录，方便故障的排查。配置方法和 `slowlog-log-slower-than` 类似，可以在配置文件中指定，也可以在命令行执行 `config set` 来设置：
```bash
config set slowlog-max-len 1000
```

### 查看慢查询日志
尽管 Redis 把慢查询日志记录到了内部的列表，但我们不能直接操作该列表，Redis 专门提供了一组命令来查询慢查询日志：
1. 获取慢查询日志：
`slowlog get [n]`
下面操作返回当前 Redis 的所有慢查询记录，可以通过参数 `n` 指定查看条数：
```bash
127.0.0.1:6379> slowlog get
 1) 1) (integer) 456
    2) (integer) 1531632044
    3) (integer) 3
    4) 1) "get"
       2) "m"
    5) "127.0.0.1:50106"
    6) ""
 2) 1) (integer) 455
    2) (integer) 1531632037
    3) (integer) 14
    4) 1) "keys"
       2) "*"
    5) "127.0.0.1:50106"
    6) ""
```
	结果说明：
`1)` 慢查询记录 id；
`2)` 发起命令的时间戳；
`3)` 命令耗时，单位为微秒；
`4)` 该条记录的命令及参数；
`5)` 客户端网络套接字（ip: port）;
`6)` ""
2. 获取当前慢查询日志记录数
`slowlog len`
```
127.0.0.1:6379> slowlog len
(integer) 458
```
3. 慢查询日志重置
`slowlog reset`
实际上是对慢查询列表做清理操作：
```bash
127.0.0.1:6379> slowlog len
(integer) 461
127.0.0.1:6379> slowlog reset
OK
127.0.0.1:6379> slowlog len
(integer) 1
```

