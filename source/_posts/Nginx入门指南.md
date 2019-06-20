---
title: Nginx入门指南
date: 2017-12-03 11:05:10
categories:
    - Nginx
tags: Nginx
---

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3766190-9339a28741405701.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# Nginx入门指南
##一.Nginx概述:
###1.什么是Nginx?
Nginx（发音同engine x）是一个网页服务器，它能反向代理HTTP, HTTPS, SMTP, POP3, IMAP的协议链接，以及一个负载均衡器和一个HTTP缓存。
起初是供俄国大型的门户网站及搜索引擎Rambler（俄语：Рамблер）使用。此软件BSD-like协议下发行，可以在UNIX、GNU/Linux、BSD、Mac OS X、Solaris，以及Microsoft Windows等操作系统中运行。
###2.Nginx的特点:
 - **更快:**
    - **单次请求会得到更快的响应**
    - **在高并发环境下,Nginx比其他web服务器有更快的响应**
 - **高扩展性:**
     - **nginx是基于模块化设计,由多个耦合度极低的模块组成,因此具有很高的扩展性。许多高流量的网站都倾向于开发符合自己业务特性的定制模块。**
 - **高可靠性:**
   - **nginx的可靠性来自于其核心框架代码的优秀设计，模块设计的简单性；另外，官方提供的常用模块都非常稳定，每个worker进程相对独立，master进程在一个worker进程出错时可以快速拉起新的worker子进程提供服务。**
 - **低内存消耗:**
  - **一般情况下，10 000个非活跃的HTTP Keep-Alive连接在Nginx中仅消耗2.5MB的内存，这是nginx支持高并发连接的基础。**
 - **单机支持10万以上的并发连接:**
  - **理论上，Nginx支持的并发连接上限取决于内存，10万远未封顶。 **
 - **热部署:**
  - **master进程与worker进程的分离设计，使得Nginx能够提供热部署功能，即在7x24小时不间断服务的前提下，升级Nginx的可执行文件。当然，它也支持不停止服务就更新配置项，更换日志文件等功能。**
 -  **最自由的BSD许可协议:**
  - **这是Nginx可以快速发展的强大动力。BSD许可协议不只是允许用户免费使用Nginx，它还允许用户在自己的项目中直接使用或修改Nginx源码，然后发布。**
###3.目前web服务器市场份额图：
![](./webserver.png)
###4.正向代理和反向代理的区别:
- **正向代理(Forward Proxy):**
  一般情况下，如果没有特别说明，代理技术默认说的是正向代理技术。关于正向代理的概念如下：
正向代理(forward)是一个位于客户端【用户A】和原始服务器(origin server)【服务器B】之间的服务器【代理服务器Z】，为了从原始服务器取得内容，用户A向代理服务器Z发送一个请求并指定目标(服务器B)，然后代理服务器Z向服务器B转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。如下图：
![](./forward.png)
- **反向代理(Reverse Proxy):**
反向代理正好与正向代理相反，对于客户端而言代理服务器就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理的命名空间(name-space)中的内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端。
**&nbsp;&nbsp;&nbsp;反向代理服务的作用:**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*1.保护和隐藏原始资源服务器:*
![](./1487527542346.png)
由于防火墙的作用，只允许代理服务器Z访问原始资源服务器B。尽管在这个虚拟的环境下，防火墙和反向代理的共同作用保护了原始资源服务器B，但用户A并不知情。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*2.负载均衡:*
![](./1487527728657.png)
当反向代理服务器不止一个的时候，我们甚至可以把它们做成集群，当更多的用户访问资源服务器B的时候，让不同的代理服务器Z（x）去应答不同的用户，然后发送不同用户需要的资源。
##二.安装Nginx:
###1.准备工作:
- **Linux操作系统:**
内核为Linux 2.6及以上的版本的操作系统，因为2.6以上的内核才支持epoll，而在linux上使用select或poll来解决事件的多路复用，是无法解决高并发压力问题的。
- **安装Nginx的必备软件:**
 - **GCC编译器**
 - **PCRE库:是用C语言写的一个正则库，目前被很软件所使用。如果我们在配置文件nginx.conf里使用了正则表达式，那么在编译Nginx时就必须把PCRE库编译进Nginx，因为Nginx的HTTP模块要靠它来解析正则表达式。yum安装: yum install -y pcre pcre-devel**
 - **zlib库：zlib库用于对HTTP包的内容做gzaip格式的压缩，如果我们在nginx.conf里配置了gzip on，并指定对于某些类型的HTTP响应使用gzip来进行压缩以减少网络传输量，那么，在编译时就必须把zlib编译进Nginx。安装方式：yum install -y zlib zlib-devel**
 - **OpenSSL开发库:如果我们的服务器不只是要支持HTTP，还要在更安全的SSL协议上传输HTTP，那么就需要拥有OpenSSL了。安装方式：yum install -y openssl openssl-devel**
 上面所列的4个库只是完成web服务器最基本功能所必须的。Nginx是模块化设计的，它的功能是由许多模块来支持的，所以如果使用了某个模块，而这个模块使用了一些第三方库，那么就必须先安装这些软件。
- **获取Nginx源码:**
 Nginx官网:[http://nginx.org](http://nginx.org)
![](./1488039019886.png)

**在此下载Nginx最新版nginx-1.11.10:**
```bash
wget http://nginx.org/download/nginx-1.11.10.tar.gz -O - | tar zxvf -
```
###2.编译安装Nginx:
 - **首先安装前面提到的软件包:**
  - **安装PCRE库:**
      ``` bash
              sudo yum install -y pcre pcre-devel
      ```

   - **安装zlib库:**
      ``` bash
          sudo yum install -y zlib zlib-devel
     ```
   - **安装OpenSSL库:**
   ``` bash
       sudo yum install -y openssl openssl-devel
   ```
运行以上命令后可以查看是否安装成功:
![](./1488087359555.png)
- **编译并安装nginx:**
Nginx源码目录介绍：
![](./1488118637317.png)
 - **进入nginx目录并执行configure命令:**
configure命令主要做一些系统相关的检测，并指定编译nginx时的一些参数，最终生成makefile供make来执行。
configure的命令参数可以通过如下命令来查看:
``` bash
./configure --help
```
![](./1488119977606.png)
   **执行configure命令，添加一些定制化参数，并生成makefile:** 
``` bash
     ./configure --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --with-debug 
```
**相关参数说明:**
   **--with-http_ssl_module:** 使nginx支持SSL协议，提供HTTPS服务。
   **--with-http_stub_status_module:**该模块可以让运行中的Nginx提供性能统计页面，获取相关的并发连接，请求信息。
 **--with-http_gzip_static_module:**在做gzip压缩前，先查看相同位置是否有已经做过gzip压缩的.gz文件，如果有，就直接返回。
 **--with-debug:**在nginx运行时通过修改配置文件来使其打印调试日志，这对于研究，定位nginx问题非常有帮助。
**configure命令执行完后，会在源码目录生成Makefile文件和一个objs目录:**
Makefile文件是指导make命令编译的脚本文件，objs目录存放编译过程中产生的二进制文件和一些configure产生的源代码文件。
**现在nginx源码目录树如下(红框圈出来的味新生成的目录):**
![](./1488127919445.png)
![](./1488128000832.png)


- **执行make命令编译:**
``` bash
make
```
- **执行make install安装:**
``` bash
sudo make install
```
###3.Nginx的使用:
- **默认方式启动nginx:**
``` bash
/usr/local/nginx/sbin/nginx
```
这时，会读取默认路径下的配置文件:/usr/local/nginx/conf/nginx.conf
- **指定自定义配置文件启动:**
``` bash
/usr/local/nginx/sbin/nginx -c path/nginx.conf
```
这时，会读取-c参数指定的配置文件来启动Nginx。
- **指定全局配置项的启动方式:**
``` bash
/usr/local/nginx/sbin/nginx -g "pid /var/nginx/test.pid;"
```
上面的命令意味着会把pid文件写入到/var/nginx/test.pid中。
- **测试配置文件是否有效:**
``` bash
/usr/local/nginx/sbin/nginx -t
```
- **在测试配置阶段不输出信息:**
``` bash
/usr/local/nginx/sbin/nginx -t -q
```
测试配置选项是时，使用-q参数可以不把error级别以下的信息输出到屏幕。
- **显示版本信息:**
``` bash
/usr/local/nginx/sbin/nginx -v
```
- **显示编译阶段的参数:**
``` bash
/usr/local/nginx/sbin/nginx -V
```
-V参数除了可以显示Nginx的版本信息外，还可以显示配置编译阶段的信息，如gcc编译器版本，操作系统版本，执行configure时的参数等。
- **快速地停止服务:**
``` bash
/usr/local/nginx/sbin/nginx -s stop
```
使用-s stop可以强制停止Nginx服务。-s参数其实是告诉Nginx程序向正在运行的Nginx服务发送信号，Nginx程序通过nginx.pid文件得到master进程的ID，再向运行中的master进程发送TERM信号来快速地关闭Nginx服务。 
**同样可以给master进程发送TERM或者INT信号来快速停止:**
```
kill -s SIGTERM NginxMasterPid
kill -s SIGINT NginxMasterPid
```
- **''优雅''地停止服务:**
``` bash
/usr/local/nginx/sbin/nginx -s quit
```
如果希望Nginx服务可以正常地处理完当前前所有请求再停止服务，那么可以使用-s quit参数来停止服务。这种方式首先会关闭监听端口，停止接收新的连接，然后把当前正则处理的连接全部处理完，最后再退出进程。
**同样可以发送信号的方式优雅地停止服务:**
``` bash
kill -s SIGQUIT NginxMasterPid
```
**优雅地停止某个worker进程：**
``` bash
kill -s SIGWINCH NginxWorkerPid
```
- **使运行中的Nginx重新加载配置文件并生效:**
``` bash
/usr/local/nginx/sbin/nginx -s reload
```
**同样可以通过kill发送信号来达到同样的效果:**
```
  kill -s SIGHUP NginxMasterPid
```
##二.Nginx的配置:
#### 1.运行中的Nginx进程间的关系:
在正式提供服务的产品环境下，部署Nginx时都是使用一个master进程来管理多个worker进程，一般情况下，worker进程的数量与服务器上的CPU核心数相等。每一个worker进程都是繁忙的，他们真正地提供互联网服务，master进程则很"清闲"，只负责监控管理worker进程。部署后的Nginx进程间的关系如图下图所示:
![](./1488387700110.png)
#### 2.Nginx配置中的通用语法:
Nginx的配置文件其实是一个普通的文本文件。下面来看一个简单的例子:
```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```
- **块配置项:**
块配置项由一个块配置名和一对大括号组成。具体示例如下:
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```
上面代码段中的events，http，server，location都是块配置项。块配置项可以嵌套，内层块直接继承外层块，例如，上例中server块里的任意配置都是基于http块里的已有配置的。当内外层中的配置发生冲突时，究竟是以内层块还是外层块的配置为准，取决于解析这个配置项的模块。
- **配置项的语法格式:**
 最基本的配置项语法格式如下:
 **配置项名 配置项值1 配置项值2 ...；**
 首先，在行首的是配置项名，这些配置项名必须是Nginx的某一个模块想要处理的，否则Nginx会认为配置文件出了非法的配置项名。配置项名输入结束后，将以空格作为分隔符。
 其次是配置项值，它可以是数字或字符串(当然也包括正则表达式)。针对一个配置项，既可以有一个值，也可以有多个值，配置项值直接仍然由空格符来分隔。
 最后，每行配置的结尾需要加上分号。
- **配置项的注释:**
  如果有一个配置项暂时需要注释掉，那么可以加 "#"注释掉这一行配置。
- **配置项的单位:**
 大部分模块遵循一些通用的规定，如指定空间大小时不用每次都定义到字节，指定时间时不用精确到毫秒。
**当指定空间大小时，可以用的单位包括：**
    - K或者k千字节
    - M或者m兆字节
 例如:
    gzip_buffer 48k;
    client_max_body_size 64M;
    
  **指定时间时，可以使用的单位包括：**
   - ms，s，m，h，d，w，M，y
 例如：
 expires 10y;
 proxy_read_timeout 600;
 client_body_timeout 2m;
- **在配置中使用变量 :**
 有些模块允许在配置中使用变量，具体示例如下:
```
	log_format main '$time_local|10.4.24.116|$request|$status|'
								                    '$remote_user|$remote_addr|$http_user_agent|$http_referer|$host|'
								                    '$bytes_sent|$request_time|$upstream_response_time|$upstream_addr|'
								                    '$connection|$connection_requests|$upstream_http_content_type|$upstream_http_content_disposition';
```
其中变量前要加$符号。需要注意的是，这种变量只有少数模块支持，并不是通用的。
**注意⚠️ :**
在执行configure命令时，我们已经把许多模块编译进Nginx中，但是否启用这些模块，一般取决于配置文件中相应的配置项。换句话说，每个Nginx模块都有自己感兴趣的配置项，大部分模块都必须在nginx.conf中读取某个配置项后才会在运行时启动。例如，只有当配置http{...}这个配置项时，ngx_http_module模块才会在Nginx中启用，其他依赖ngx_http_module的模块才能正常使用。
#### 3.一个静态Web服务器常用的配置:
静态Web服务器的主要功能由ngx_http_core_module模块(HTTP框架的主要成员)实现，当然，一个完整的静态Web服务器还有许多功能是由其他的HTTP模块实现的。一个典型的静态Web服务器还会包含多个server块和location块，例如:
```
user  web;
worker_processes  6;

#error_log  logs/error.log;
error_log  logs/error.log  info;
#error_log  logs/error.log  info;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;


    log_format main '$time_local|10.4.24.116|$request|$status|'
                    '$remote_user|$remote_addr|$http_user_agent|$http_referer|$host|'
                    '$bytes_sent|$request_time|$upstream_response_time|$upstream_addr|'
                    '$connection|$connection_requests|$upstream_http_content_type|$upstream_http_content_disposition';
    
    access_log  logs/access.log  main;

    sendfile        on;
    tcp_nodelay     on;
    #tcp_nopush     on;

    keepalive_timeout  120 100;
    resolver_timeout   15;
    client_body_timeout        20;
    client_header_timeout      20;
    client_body_buffer_size 512k;

    large_client_header_buffers 1 512k;
    #gzip  on;
    
    #APP
    include server/zapier.chime.me.conf;
    include server/https_app116.chime.me.conf;
    include server/http_app116.chime.me.conf;
    #CRM 
    include server/https_test.chime.me.conf;
    include server/https_dev.chime.me.conf;
    include server/https_dev2.chime.me.conf;
    include server/https_static.chime.me.conf;
    include server/crmnginx.chime.me.conf;
    include server/brokernginx.chime.me.conf;

    include server/http_oldlender.chime.me.conf;

    include server/frontend.chime.me.conf;
    include server/chime.me.conf;
    include server/twilio.chime.me.conf;

    include server/http_testzillow.chime.me.conf; 

    #agnet 资历收集页图片地址 
    include server/uploadfile.conf;
    
    #测试nginx status
    server {
        listen *:80 ;
        server_name localhost status.chime.me;
        
        location /ngx_status {
            stub_status on;
            access_log off;
            allow 10.2.204.166;
            deny all;
        }
    }
}
```
所有的HTTP配置项必须直属于http块，location块，upstream块或if块。
**Web服务器常用配置项:**
- **虚拟主机与请求的分发:**
  由于IP地址的数量有限，因此经常会存在多个主机域名对应着同一个IP地址的情况，这时在nginx.conf中就可以按照server_name并通过server块来定义虚拟主机，每个server块就是一个虚拟主机，它只处理与之相对应的主机域名请求。这样，一台服务器上的Nginx就能以不同的方式处理访问不同主机域名的HTTP请求了。
   - **监听端口:**
   默认:listen 80;
   配置块:server
   listen参数决定Nginx服务如何监听端口。在listen后可以只加IP地址，端口活主机名，非常灵活。
   - **主机名称:**
   语法: server_name name [...];
   默认:server_name "";
   server_name后可以跟多个主机名称，如server_name www.testweb.com download.test.com;
   在开始处理一个HTTP请求时，Nginx会取出header头重的Host，与每个server中的server_name进行匹配，以决定到底由哪一个server块来处理这个请求。有可能一个Host与多个server块中的server_name都匹配，这时就会根据匹配优先级来选择实际处理的server块。
   - **location:**
   语法:location[=|~|~*|^~|@] /uri/ {...}
   配置块:server
   location会尝试根据用户请求中的URI来匹配上面的/uri表达式，如果可以匹配，就选择location {}块中的配置来处理用户请求。
- **文件路径的定义:**
   - **以root方式设置资源路径:**
   语法:root path
   默认:root html
   配置块:http，server，location，if
   例如，定义资源文件相对于HTTP请求的根目录。
```
   location /download/ {
       root /opt/web/html/;
   }
```
 在上面配置中，如果有一个请求的URI是/download/index/test.html，那么web服务器将会返回服务器上/opt/web/html/download/index/test.html
   - **访问首页:**
     语法:index file ...;
     默认:index index.html;
     配置块:http,server,location
     有时访问站点的URI是/，这时一般返回网站的首页。这里用ngx_http_index_module模块提供的index配置实现。index后可以跟多个文件参数，Nginx将会按照顺序来访问这些文件，例如:
```
     location / {
        root path;
        index /index.html /html/inde.php /index.php
     }
```
  接受到请求后，Nginx首先会尝试访问path/index.php文件，如果可以访问，就直接返回文件内容结束请求，否则再试图返回path/html/index.php的内容，依此类推。
 - **网络连接的设置:**
   - **读取HTTP头部的超时时间:**
    语法:client_header_timeout time (默认单位:秒)
    默认:client_header_timeout 60;
    配置块:http, server, location
    客户端与服务器建立连接后将开始接收HTTP头部，在这个过程中，如果在一个时间间隔内没有读取到客户端发来的字节，则认为超时，并向客户端返回408("Rquest timed out")响应。
    - **读取HTTP头部的超时时间:**
     语法:client_body_timeout time (默认单位:秒)
     默认:client_body_timeout 60;
     配置块:http, server, location
     此配置项与client_header_timeout相似，只是这个超时时间只在读取HTTP包体时才有效。
    - **发送响应的超时时间:**
     语法:send_timeout time;
     默认:send_timeout 60;
     这个超时时间时发送响应的超时时间，即Nginx服务器向客户端发送了数据包，但客户端一直没有去接收这个数据包。如果某个连接超过send_timeout定义的超时时间，那么Nginx将会关闭这个连接。
    - **keepalive超时时间:**
     语法:keepalive_timeout time (默认单位:秒)
     默认:keepalive_timeout 75
     配置块:http, server, location
     一个keepalive_timeout连接在闲置超过一段时间后，服务器和浏览器都会去关闭这个连接。当然，keepalive_timeout配置项是用来约束Nginx服务器的，Nginx也会按照规范把这个时间传个浏览器，但每个浏览器对待keepalive的策略有可能是不同的。
#### 4.配置一个静态web服务器:
nginx的默认配置就是一个静态web服务器，启动后可以直接访问，配置文件如下:
```
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```  
#### 5.给Nginx的access log添加唯一的request id字段:
- **1.下载uuid-perl模块:**
```
wget http://mirror.centos.org/centos/7/os/x86_64/Packages/uuid-perl-1.6.2-26.el7.x86_64.rpm
```
- **2.yum安装上面下载的rpm包，自动解决依赖:**
```
yum install uuid-perl-1.6.2-26.el7.x86_64.rpm
```
- **3.重新编译ngixn，添加perl模块:**
 首先安装perl依赖库，要不然报错:
```
 sudo yum install -y perl-devel perl-ExtUtils-Embed
```
 然后配置编译参数，执行configure生成makfile:
```
 ./configure --with-http_ssl_module --with-http_stub_status_module --with-http_gzip_static_module --with-debug --with-http_perl_module
```
- **4.修改nginx配置文件:**
 ![](./1488432652846.png)
- **5.重新加载配置文件即可生效:**
```
 sbin/nginx -s reload
```
- **6.效果如下:**
![](./1488432839394.png)

