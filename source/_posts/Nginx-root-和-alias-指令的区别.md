---
title: Nginx root 和 alias 指令的区别
date: 2019-05-31 12:25:00
categories: Nginx
tags: Nginx
---

nginx 的 root 和 alias 指令都是用于访问服务器本地文件的，两者区别如下：
- 配置语法及适用范围
```
[root]
语法：root path
默认值：root html
配置段：http、server、location、if

[alias]
语法：alias path
配置段：location
```

- root 的处理结果是：root 路径＋location 路径；
- alias 的处理结果是：使用 alias 路径替换 location 路径；
- alias 后面的路径结尾必须是 '/'，而 root 可有可无；

#### 举例
##### 当 location 配置为如下时：
```
location ^~ /documents/ {
    root /var/www-html/documents/;
}
```
请求：GET /documents/a.js  ---> 相当于请求本地路径：/var/www-html/documents/documents/a.js

请求：GET /documents/html/index.html  ---> 相当于请求本地路径：/var/www-html/documents/documents/html/index.html

##### 当 location 配置为如下时：
```
location ^~ /documents/ {
  alias /var/www-html/documents/;
}
```
请求：GET /documents/a.js  ---> 相当于请求本地路径：/var/www-html/documents/a.js

请求：GET /documents/html/index.html  ---> 相当于请求本地路径：/var/www-html/documents/html/index.html
