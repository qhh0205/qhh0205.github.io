---
title: 基于 Docker Compose 容器化搭建 Wordpress
date: 2019-01-14 22:57:55
categories: Wordpress
tags: Wordpress
---

最近由于业务需求帮公司搞了几个 Wordpress 作为官网，中间也是踩了不少坑，倒不是搭建 wordpress 难，主要是 wordpress 本身坑就挺多的，比如迁移、使用过程中文件上传大小的限制问题、迁移后域名无法变更问题等等。

接下来演示如何基于 Docker Compose 来容器化搭建一个可靠、易维护的 Wordpress 网站，可靠指的是服务挂了会自愈（当然是 docker 本身的功能了），易维护指的是即使后面做服务的迁移也是非常方便的，只是简单的文件拷贝，然后 docker compose 启动，没有任何其他的维护成本。

**架构**：非容器化 nginx 反向代理 + Docker Compose （ Wordpress + MySql）

![](/images/nginx-wordpress-docker.png)

### Docker Compose 工程
Wordpress Docker Compose 工程目录结构：
```
wordpress
├── db_data  # mysql 数据目录
├── docker-compose.yaml  # docker-compose 文件
├── upload.ini  # php 文件上传相关配置
└── wp_site  # wordpress 静态资源存储目录
```
docker-compose.yaml:
```yaml
version: '3.3'

services:
   db:
     image: mysql:5.7
     volumes:
       - ./db_data:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: example

   wordpress:
     depends_on:
       - db
     image: wordpress:5.0.3
     volumes:
       - ./wp_site:/var/www/html
       - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
     ports:
       - "9000:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: example
```
uploads.ini:
```yaml
file_uploads = On
memory_limit = 128M
upload_max_filesize = 512M
post_max_size = 128M
max_execution_time = 600
```

### 外部 Nginx  配置文件
https_server.conf（网站配置文件）:
```
server {
        listen      80;
        server_name example.com;
        rewrite ^(.*)$  https://$host$1 permanent;
}

server {
        listen      443;
        ssl on;
        ssl_certificate crts/example/example_com.crt;
        ssl_certificate_key crts/example/example_com.key;
        server_name example.com;
        location / {
                proxy_pass   http://localhost:9002;
                include conf.d/common.cfg;
                proxy_set_header X-Forwarded-Proto https;
        }
}
```
common.cfg（Nginx 相关配置项）:
```
proxy_set_header   Host             $host;
proxy_set_header   X-Real-IP        $remote_addr;
proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
client_max_body_size       128m;
client_body_buffer_size    10m;
client_body_temp_path      /tmp/client_body_temp;
proxy_connect_timeout      40;
proxy_send_timeout         20;
proxy_read_timeout         20;
proxy_buffer_size          256k;
proxy_buffers              32 64k;
```

### 启动服务
进入 docker compose 工程目录执行：
```bash
docker-compose up -d
```

### Tips
相关 docker compose 指令：
`docker-compose stop`: 停止已启动的服务，停止后容器还在，只是退出了；
`docker-compose start`: 启动已停止的服务；
`docker-compose down`: 停止并清理掉启动的 Docker 容器、卷、网络等相关资源；
`docker-compose logs -f`: 实时查看日志
