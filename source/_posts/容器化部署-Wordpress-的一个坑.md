---
title: 容器化部署 Wordpress  的一个坑
date: 2018-10-13 23:46:09
categories: Wordpress
tags: Wordpress
---

### 问题描述
非容器化 nginx + docker-compose 容器化 wordpress 后，媒体库上传图片报错：HTTP 错误

![](/images/wp-http-error.png)
### 问题解决
其实这个问题的原因非常多，网上文章一大堆（https://www.duoluodeyu.com/2402.html ），但是本文中所遇到同样问题的原因却比较诡异：*nginx client_max_body_size 参数必须要和 PHP 的 post_max_size 参数值一致。*

#### 1.修改 Wordpress 容器 PHP 参数
新建 uploads.ini 文件，将该文件挂载到容器：/usr/local/etc/php/conf.d/uploads.ini 文件
uploads.ini：
```
file_uploads = On
memory_limit = 128M
upload_max_filesize = 512M
post_max_size = 128M
max_execution_time = 600
```

docker-compose 文件添加卷，将文件挂载到容器
```
volumes:
       - ./wp_site:/var/www/html
       - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
```

#### 2. 修改 nginx client_max_body_size 参数配置
这个是坑的地方，这个参数的值必须要和上一步 PHP post_max_size 参数的值一致，否则还是报同样的 HTTP 错误。之前没注意这个问题，按照网上各种配置调整，均不起作用，后来经过各种猜测测试，其实问题的根因就在这里：nginx client_max_body_size 参数必须要和 php post_max_size 参数的值一致。

### 附件（完整的 Wordpress docker-compose.yaml）
容器外挂文件 uploads.ini 是定义 PHP 的一些参数配置，比如最大文件上传大小、POST 请求体大小限制、内存大小限制等等，这个文件挂载是可选的，但是如果要自定义 PHP 参数可以这么做。
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
       MYSQL_PASSWORD: wordpress
   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     volumes:
       - ./wp_site:/var/www/html
       - ./uploads.ini:/usr/local/etc/php/conf.d/uploads.ini
     ports:
       - "9001:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
```

### 相关参考文档
https://www.duoluodeyu.com/2402.html
https://github.com/docker-library/wordpress/issues/10
