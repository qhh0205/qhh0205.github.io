---
title: docker-compose 启动 Redis 服务
date: 2019-05-31 18:20:16
categories: Redis
tags: Redis
---

使用 docker-compose 以 aof 持久化方式启动单节点 Redis。
docker-compose.yml:
```yaml
---
version: '3'
services:
  redis:
    image: redis:4.0.13 
    container_name: redis
    restart: always
    command: --appendonly yes
    ports:
      - 6379:6379
    volumes:
      - ./redis_data:/data
```
