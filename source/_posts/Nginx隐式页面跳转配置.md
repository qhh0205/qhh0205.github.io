---
title: Nginx隐式页面跳转配置
date: 2018-02-18 20:58:50
categories: Nginx
tags: Nginx
---

Nginx实现将请求跳转到另一个网站的页面，在浏览器中URL保持不变。以下配置示例将请求路径https://abc.com/home/test 跳转到https://def.com/home/test/test.html 页面。
```bash
server {
    listen       443;
    server_name  abc.com;   
    include server/ssl.conf;

    location = /home/test {
        rewrite /home/test /home/test/test.html break;
        proxy_pass https://def.com;
    }
}
```
