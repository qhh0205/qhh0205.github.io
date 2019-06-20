---
title: 手动进行主机DNS配置
date: 2017-12-09 21:07:52
categories:
    - DNS
tags: DNS
---

本文介绍了如何手动进行主机DNS配置，有时候机器服务提供商不提供DHCP服务，或者由于错误的DNS配置导致服务器无法解析域名，此时就需要检查下服务器的DNS配置并配置正确的nameserver，首先介绍几个和DNS有关的配置文件：
* **/etc/hosts：**本地域名到IP的映射文件，一般默认Linux域名与IP的对应解析以此文件优先；
* **/etc/resolv.conf：**ISP的DNS服务器IP地址，本文件定义了本机进行域名解析请求的地址；
* **/etc/nsswitch.conf：**这个文件则是来决定先要使用/etc/hosts还是/etc/resolv.con的配置。文件中hosts字段定义了优先使用哪个文件来进行DNS解析，其中"files"就是使用/etc/hosts，而后面的"dns"则是使用/etc/resolv.conf的DNS服务器来进行解析：
``` bash
#hosts:     db files nisplus nis dns
hosts:      files dns myhostname
```
一般情况下是不需要自己手动修改主机DNS的配置的，除非机器提供商不提供DHCP服务，或者本机DNS配置有问题，导致无法进行域名解析才需要手动配置，以下是配置方法：
>修改/etc/resolv.conf，添加能访问通的DNS服务器地址，以下示例修改成Google的DNS服务器地址：
``` bash
nameserver 8.8.8.8 #主DNS地址
nameserver 8.8.4.4 #备用DNS地址,在主的失效时启用备的,当然还可以添加更多的地址
```
⚠️注意：尽量不要设置超过3台以上的DNS IP在/etc/resolv.conf中，如果是你的局域网出问题，会导致无法连接到DNS服务器，那么你的主机还是会向每台DNS服务器发出连接请求，每次连接都有timeout时间的等待，会导致浪费非常多的时间。
