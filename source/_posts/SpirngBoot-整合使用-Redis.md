---
title: SpirngBoot 整合使用 Redis
date: 2021-07-17 23:24:16
categories: Java
tags: Java
---


本文主要介绍如何在 springboot 项目中集成使用 Redis。springboot 将很多基础的工具组件都封装成了一个个的 starter，比如基础的 web 框架相关的 pring-boot-starter-web，操作数据库相关的 spring-boot-starter-data-jpa 等等。如果要操作 Redis，同理需要引入 redis 相关的 starter：spring-boot-starter-data-redis。下面介绍 springboot 集成使用 Redis 的详细过程。

### 1、引入 redis starter POM 依赖
```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### 2、Redis 数据源配置
springboot 配置文件（application.properties）添加如下 redis 配置：
```
# Redis服务器地址
spring.redis.host=127.0.0.1
# Redis服务器连接端口
spring.redis.port=6379
# Redis服务器连接密码（默认为空）
spring.redis.password=
```

### 3、使用示例
写一个测试用例测试使用下，添加并查询 K-V：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisUsageTest extends TestCase {
    @Autowired
    private StringRedisTemplate redisTemplate;

    @Test
    public void redisUsageTest() {
        redisTemplate.opsForValue().set("name", "jim");
        System.out.println(redisTemplate.opsForValue().get("name"));
    }
}
```

登录 Redis 控制台可以看到数据已经写入了 Redis
```
127.0.0.1:6379> get name
"jim"
127.0.0.1:6379>
```

### 参考文档
[https://cloud.tencent.com/developer/article/1457454](https://cloud.tencent.com/developer/article/1457454)
[https://blog.csdn.net/tuzongxun/article/details/107794207](https://cloud.tencent.com/developer/article/1457454)


