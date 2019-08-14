---
title: 基于 docker-compose 容器化构建 Kong 微服务网关平台
date: 2019-08-14 20:05:23
categories: 微服务网关
tags: 微服务网关
---

本文主要介绍如何使用 docker-compose 快速体验 Kong 微服务网关，先简单介绍基本概念，然后做了一个 Demo 测试使用，涉及到的相关话题有：
- Kong 简介；
- Konga 简介；
- 基于 docker-compose 容器化构建 Kong 微服务网关平台；
- 使用 Konga 可视化创建一个 Service 及路由；

### Kong 简介
[Kong](https://konghq.com/kong/) 是微服务网关模式架构中连接服务消费方和服务提供方的中间件系统，即我们经常所说的微服务网关，网关将各自的业务系统的演进和发展做了天然的隔离，使业务系统更加专注于业务服务本身，同时微服务网关还可以为服务提供和沉淀更多附加功能。微服务网关的主要作用如下：
- 请求接入：管理所有请求接入，作为所有 API 接口的请求入口；
- 业务聚合：所有微服务后端可以注册在 API 网关，通过 API 网关统一暴露服务；
- 拦截策略：可以通过统一的安全、路由、流控等公共服务组件；
- 统一管理：提供统一的监控工具，配置管理工具等基础设施；

Kong 作为完全开源的微服务网关，基于 Nginx 实现，所以其性能表现是毋庸置疑的，另外和其他网关系统类似，也支持插件化扩展，提供了丰富的插件可供使用。Kong 进程启动后会启动多个端口，每个端口功能也不一样：
- 8001 端口：http 管理 API；
- 8444 端口：https 管理 API；
- 8000 端口：接收处理 http 流量；
- 8443 端口：接收处理 https 流量；

![](/images/kong.png)

Kong 的使用特别简单，需要搞懂几个概念就可以快速使用了：
- Service：Service 是要对外暴露的上游服务，类似于 Nginx 反向代理配置的 upstream；
- Route：Route 定义了路由规则，外部流量如何路由到相应的 Service；
- Consumer：类似账号的概念，可以设置不同的 Consumer 对 API 的访问限制；

关于 Kong 数据持久化
Kong 有两种运行模式，以 [db-less](https://docs.konghq.com/1.2.x/db-less-and-declarative-config/) 模式运行时所有路由、Service 等信息都存储在内存中，这些信息都是通过 declarative 配置文件动态生成，然后加在到内存。另一种是以 db 模式运行，需要额外的数据库支持，用于存储路由、Service 等信息，这种方式是生产环境推荐的方式。Kong 支持两种数据库持久化：Postgres 或者 Cassandra。

### Konga 简介
[Konga](https://github.com/pantsel/konga) 是一个第三方开源的针对 Kong 网关的 UI 管理界面，与 Kong 没有关系。通过 Konga 我们可以可视化配置 Kong 相关的配置，在没有可视化界面的情况下只能通过 curl 调用 Kong 提供的 Admin API 来管理 Kong 配置，相对于可视化配置来说复杂度是显而易见的。Konga 支持的主要特性如下：
- 管理所有 Kong Admin API 对象；
- 多个 Kong 节点管理；
- 使用快照备份、恢复 Kong 节点；
- 通过健康检查健康节点和 API 状态；
- 支持邮寄和 Slack 告警通知；
- 多用户管理；
- 支持数据库集成（MySQL，postgresSQL，MongoDB，SQL Server）；

Konga 默认将用户信息和配置信息存在在本地磁盘文件，我们可以选择和数据库集成，将相关信息存储到数据库，这也是生产环境推荐的做法。

### 使用 docker-compose 容器化构建 Kong 微服务网关平台
使用 docker-compose 将 Kong 网关、Konga UI 管理页面、数据库三个服务组合起来，组成一个完整、可用的网关系统，Kong 网关和 Konga 服务共用一个 postgres 数据库。下面为启动整个系统完整的 docker-compose 文件，来自：https://gist.github.com/pantsel/73d949774bd8e917bfd3d9745d71febf 在其基础上对存在的问题进行了修复：
docker-compose.yml
```yaml
version: "3"

networks:
 kong-net:
  driver: bridge

services:

  #######################################
  # Postgres: The database used by Kong
  #######################################
  kong-database:
    image: postgres:9.6
    restart: always
    networks:
      - kong-net
    environment:
      POSTGRES_USER: kong
      POSTGRES_DB: kong
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "kong"]
      interval: 5s
      timeout: 5s
      retries: 5

  #######################################
  # Kong database migration
  #######################################
  kong-migration:
    image: kong:latest
    command: "kong migrations bootstrap"
    networks:
      - kong-net
    restart: on-failure
    environment:
      KONG_PG_HOST: kong-database
    links:
      - kong-database
    depends_on:
      - kong-database

  #######################################
  # Kong: The API Gateway
  #######################################
  kong:
    image: kong:latest
    restart: always
    networks:
      - kong-net
    environment:
      KONG_PG_HOST: kong-database
      KONG_PROXY_LISTEN: 0.0.0.0:8000
      KONG_PROXY_LISTEN_SSL: 0.0.0.0:8443
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    depends_on:
      - kong-migration
      - kong-database
    healthcheck:
      test: ["CMD", "curl", "-f", "http://kong:8001"]
      interval: 5s
      timeout: 2s
      retries: 15
    ports:
      - "8001:8001"
      - "8000:8000"
      - "8443:8443"

  #######################################
  # Konga database prepare
  #######################################
  konga-prepare:
    image: pantsel/konga:next
    command: "-c prepare -a postgres -u postgresql://kong@kong-database:5432/konga_db"
    networks:
      - kong-net
    restart: on-failure
    links:
      - kong-database
    depends_on:
      - kong-database

  #######################################
  # Konga: Kong GUI
  #######################################
  konga:
    image: pantsel/konga:latest
    restart: always
    networks:
        - kong-net
    environment:
      DB_ADAPTER: postgres
      DB_HOST: kong-database
      DB_USER: kong
      TOKEN_SECRET: km1GUr4RkcQD7DewhJPNXrCuZwcKmqjb
      DB_DATABASE: konga_db
      NODE_ENV: production
    depends_on:
      - kong-database
    ports:
      - "1337:1337"
```
docker-compose 一键启动相关服务：
```bash
docker-compose up -d
```
启动后访问 http://nodeIP:1337 即可看到 Konga 登陆注册页面，首次访问需要注册用户，随便填写用户名称、邮箱等相关信息即可：

![](/images/konga1.png)

在登陆后页面比较简单，因为还没有连接到 Kong，下面配置连接到要管理的 Kong 节点：
Name：随便填写；
Kong Admin URL：填写 Kong Admin API 地址；

![](/images/konga2.png)
连接到 Kong 节点后即可看到节点更详细的信息：
 
![](/images/konga3.png)

### 使用 Konga 可视化创建一个 Service 及路由
Kong 官方文档给了一个 Kong 的使用 [例子](https://docs.konghq.com/1.2.x/getting-started/configuring-a-service/)：通过 curl 调用 Kong Admin API 创建一个 Service 和路由，然后通过 curl 测试访问。这里我们演示如何通过 Konga 可视化做同样的配置：

首先创建一个 Service，指向 http://mockbin.org 服务：

![](/images/konga4.png)
![](/images/konga5.png)

创建一个对外访问的路由，路由到上一步创建的 Service：

![](/images/konga6.png)
![](/images/konga7.png)
![](/images/konga8.png)

上面配置中 Hosts 即为对外访问的 host 名称，只要将 api.example.com 绑定到网关 IP 地址即可进行访问，会路由到绑定的 Service：
```bash
curl http://api.example.com:8000
```

### 相关文档
https://docs.konghq.com/ | Kong 官方文档
https://github.com/pantsel/konga/ | Konga GitHub 仓库
https://gist.github.com/pantsel/73d949774bd8e917bfd3d9745d71febf | kong docker-compose

