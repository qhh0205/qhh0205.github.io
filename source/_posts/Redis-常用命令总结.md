---
title: Redis 常用命令总结
date: 2019-01-19 13:54:46
categories: Redis
tags: Redis
---

## Redis 常用命令总结
### redis-cli
redis-cli 是 redis 的客户端工具，有很多实用的参数。

![](/images/redis-cli.png)

### redis-benchmark
redis-benchmark 为 redis 提供的性能测试工具，对 redis 各种数据的操作进行测试，并给出测试结果。如下为 GET 操作的测试报告样例：
```
====== GET ======
  20000 requests completed in 0.36 seconds
  100 parallel clients
  3 bytes payload
  keep alive: 1

62.01% <= 1 milliseconds
97.57% <= 2 milliseconds
99.99% <= 3 milliseconds
100.00% <= 3 milliseconds
55865.92 requests per second
```
![](/images/redis-benchmark.png)

