---
title: SpringBoot 使用 Redis 分布式锁解决并发问题
date: 2021-07-24 17:54:34
categories: Java
tags: Java
---

### 问题背景
现在的应用程序架构中，很多服务都是多副本运行，从而保证服务的稳定性。一个服务实例挂了，其他服务依旧可以接收请求。但是服务的多副本运行随之也会引来一些分布式问题，比如某个接口的处理逻辑是这样的：接收到请求后，先查询 DB 看是否有相关的数据，如果没有则插入数据，如果有则更新数据。在这种场景下如果相同的 N 个请求并发发到后端服务实例，就会出现重复插入数据的情况：
![Alt text](/images/distribute-issue.jpg)

### 解决方案
针对上面问题，一般的解决方案是使用分布式锁来解决。同一个进程内的话用本进程内的锁即可解决，但是服务多实例部署的话是分布式的，各自进程独立，这种情况下可以设置一个全局获取锁的地方，各个进程都可以通过某种方式获取这个全局锁，获得到锁后就可以执行相关业务逻辑代码，没有拿到锁则跳过不执行，这个全局锁就是我们所说的分布式锁。分布式锁一般有三种实现方式：1. 数据库乐观锁；2. 基于Redis的分布式锁；3. 基于ZooKeeper的分布式锁。

我们这里介绍如何基于 Redis 的分布式锁来解决分布式并发问题：Redis 充当获取全局锁的地方，每个实例在接收到请求的时候首先从 Redis 获取锁，获取到锁后执行业务逻辑代码，没争抢到锁则放弃执行。
![Alt text](/images/redis-lock-impl.png)

**主要实现原理：**
Redis 锁主要利用 Redis 的 setnx 命令：

加锁命令：SETNX key value，当键不存在时，对键进行设置操作并返回成功，否则返回失败。KEY 是锁的唯一标识，一般按业务来决定命名。Value 一般用 UUID 标识，确保锁不被误解。

解锁命令：DEL key，通过删除键值对释放锁，以便其他线程可以通过 SETNX 命令来获取锁。

锁超时：EXPIRE key timeout, 设置 key 的超时时间，以保证即使锁没有被显式释放，锁也可以在一定时间后自动释放，避免资源被永远锁住。

**可靠性：**
为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下四个条件：

- 互斥性。在任意时刻，保证只有一台机器的一个线程可以持有锁；
- 不会发生死锁。即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁；
- 具备非阻塞性。一旦获取不到锁就立刻返回加锁失败；
- 加锁和解锁必须是同一个客户端，客户端自己不能把别人加的锁给解了；

###  SpringBoot 集成使用 Redis 分布式锁
写了一个 RedisLock 工具类，用于业务逻辑执行前加锁和业务逻辑执行完解锁操作。这里的加锁操作可能实现的不是很完善，有加锁和锁过期两个操作原子性问题，如果 SpringBoot 版本是2.x的话是可以用注释中的代码在加锁的时候同时设置锁过期时间，如果 SpringBoot 版本是2.x以下的话建议使用 Lua 脚本来确保操作的原子性，这里为了简单就先这样写：
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

/**
 * @description: Redis分布式锁实现工具类
 * @author: qianghaohao
 * @time: 2021/7/19
 */
@Component
public class RedisLock {
    @Autowired
    StringRedisTemplate redisTemplate;

    /**
     * 获取锁
     *
     * @param lockKey    锁
     * @param identity   身份标识（保证锁不会被其他人释放）
     * @param expireTime 锁的过期时间（单位：秒）
     * @return
     */
    public boolean lock(String lockKey, String identity, long expireTime) {
        // 由于我们目前 springboot 版本比较低，1.5.9，因此还不支持下面这种写法
        // return redisTemplate.opsForValue().setIfAbsent(lockKey, identity, expireTime, TimeUnit.SECONDS);
        if (redisTemplate.opsForValue().setIfAbsent(lockKey, identity)) {
            redisTemplate.expire(lockKey, expireTime, TimeUnit.SECONDS);
            return true;
        }
        return false;
    }

    /**
     * 释放锁
     *
     * @param lockKey  锁
     * @param identity 身份标识（保证锁不会被其他人释放）
     * @return
     */
    public boolean releaseLock(String lockKey, String identity) {
        String luaScript = "if " +
                "  redis.call('get', KEYS[1]) == ARGV[1] " +
                "then " +
                "  return redis.call('del', KEYS[1]) " +
                "else " +
                "  return 0 " +
                "end";
        DefaultRedisScript<Boolean> redisScript = new DefaultRedisScript<>();
        redisScript.setResultType(Boolean.class);
        redisScript.setScriptText(luaScript);
        List<String> keys = new ArrayList<>();
        keys.add(lockKey);
        Object result = redisTemplate.execute(redisScript, keys, identity);
        return (boolean) result;
    }
}
```

###  使用示例
这里只贴出关键的使用代码，注意：锁的 key 根据自己的业务逻辑命名，能唯一标示同一个请求即可。value 这里设置为 UUID，为了确保释放锁的时候能正确释放（只释放自己加的锁）。
```java
@Autowired
private RedisLock redisLock;  // redis 分布式锁
```
```java
        String redisLockKey = String.format("%s:docker-image:%s", REDIS_LOCK_PREFIX, imageVo.getImageRepository());
        String redisLockValue = UUID.randomUUID().toString();
        try {
            if (!redisLock.lock(redisLockKey, redisLockValue, REDIS_LOCK_TIMEOUT)) {
                logger.info("redisLockKey [" + redisLockKey + "] 已存在，不执行镜像插入和更新");
                result.setMessage("新建镜像频繁，稍后重试，锁占用");
                return result;
            }
            ... // 执行业务逻辑
       catch (Execpion e) {
            ... // 异常处理
       } finally {  // 释放锁
            if (!redisLock.releaseLock(redisLockKey, redisLockValue)) {
                logger.error("释放redis锁 [" + redisLockKey + "] 失败);
            } else {
                logger.error("释放redis锁 [" + redisLockKey + "] 成功");
            }
        }
```

### 参考文档
[https://www.jianshu.com/p/6c2f85e2c586](https://www.jianshu.com/p/6c2f85e2c586)
[https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/](https://xiaomi-info.github.io/2019/12/17/redis-distributed-lock/)

