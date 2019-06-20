---
title: 使用 ansible 模板化 haproxy 配置文件
date: 2019-06-11 22:27:08
categories: Ansible
tags: Ansible
---

今天使用 ansible 自动化一些日常工作，其中包括 haproxy 的配置变更，我们 haproxy 里面定义了很多 frontend 和 backend，猛一看还不好模版化，其实仔细研究一下发现完全可以通过模板的循环语法动态生成配置文件，在此分享下。首先看一下未模板化时的原始配置：
`haproxy.cfg`:
```yaml
global
    maxconn         20000
    ulimit-n        204800
    log             127.0.0.1 local3
    user             haproxy
    group            haproxy
    chroot          /var/empty
    nbproc      4 #启动后运行的进程数量
    daemon        #以后台形式运行haproxy
    pidfile     /var/run/haproxy.pid
defaults
    log global
    mode tcp
    retries 3 #3次连接失败认为服务不可用，也可以在后面设置
    timeout connect 5s #连接超时
    timeout client 30s #客户端超时
    timeout server 30s #服务器端超时
    option redispatch
    option    nolinger
    no option dontlognull
    option    tcplog
    option log-separate-errors

listen admin_stats  #监控页面设置
    bind 0.0.0.0:26000
    bind-process 1
    mode http
    log 127.0.0.1 local3 err
    stats refresh 30s #每隔30秒自动刷新监控页面
    stats uri /admin
    stats realm welcome login\ Haproxy
    stats auth admin:123456
    stats hide-version
    stats admin if TRUE

# 通过配置文件动态生成 hproxy 配置
frontend redis
  bind *:6379
  default_backend redis
backend redis
  server 10.1.1.1:6379 10.1.1.1:6379 check inter 1500 rise 3 fall 3
frontend es
  bind *:9300
  default_backend es
backend es
  server 10.1.1.2:9300 10.1.1.2:9300 check inter 1500 rise 3 fall 3
frontend mysql
  bind *:3306
  default_backend mysql
backend mysql
  server 10.1.1.3:3306 10.1.1.3:3306 check inter 1500 rise 3 fall 3
```

观察可以发现 frontend 和 backend 是成对出现的，一对为一个完整的配置，所以可以将  frontend 和 backend 对抽象为一个变量列表的元素，我们通过定义一个 ansible 变量列表循环生成同样的配置即可。变量具体定义如下（haproxy_servers 变量），下面为一个完整的测试 playbook。
`playbook.yml`:
```yaml
---
- name: Test Playbook...
  hosts: all
  become: yes
  gather_facts: no
  vars:
    haproxy_servers:
      - frontend: 'redis'
        bind_port: 6379
        backend:
          - address: 10.1.1.1:6379
      - frontend: 'es'
        bind_port: 9300
        backend:
          - address: 10.1.1.2:9300
      - frontend: 'mysql'
        bind_port: 3306
        backend:
          - address: 10.1.1.3:3306
  tasks:
    - name: Generate haproxy config
      template:
        src: haproxy.j2
        dest: /tmp/haproxy.cfg
        mode: 0644
        force: yes
```

接下来我们看看模板文件 haproxy.j2 的定义：主要通过 jinja2 模板的循环语法遍历 haproxy_servers 变量生成 haproxy 配置。
`haproxy.j2`:
```python
global
    maxconn         20000
    ulimit-n        204800
    log             127.0.0.1 local3
    user             haproxy
    group            haproxy
    chroot          /var/empty
    nbproc      4 #启动后运行的进程数量
    daemon        #以后台形式运行haproxy
    pidfile     /var/run/haproxy.pid
defaults
    log global
    mode tcp
    retries 3 #3次连接失败认为服务不可用，也可以在后面设置
    timeout connect 5s #连接超时
    timeout client 30s #客户端超时
    timeout server 30s #服务器端超时
    option redispatch
    option    nolinger
    no option dontlognull
    option    tcplog
    option log-separate-errors

listen admin_stats  #监控页面设置
    bind 0.0.0.0:26000
    bind-process 1
    mode http
    log 127.0.0.1 local3 err
    stats refresh 30s #每隔30秒自动刷新监控页面
    stats uri /admin
    stats realm welcome login\ Haproxy
    stats auth admin:123456
    stats hide-version
    stats admin if TRUE

# 通过配置文件动态生成 hproxy 配置
{% for frontend in haproxy_servers %}
frontend {{ frontend.frontend }}
  bind *:{{ frontend.bind_port }}
  default_backend {{ frontend.frontend }}
backend {{ frontend.frontend }}
{% for backend in frontend.backend %}
  server {{ backend.address }} {{ backend.address }} check inter 1500 rise 3 fall 3
{% endfor %}
{% endfor %}
```

测试下模板化结果？：我一般用 vagrant + ansible 测试 ansible 脚本，所以直接执行 `vagrant rsync && vagrant provision` 即可看到效果。关于  vagrant + ansible 的最佳实践请戳之前的这篇文章：[使用 Vagrant 调试 Ansible Playbook](https://qhh.me/2019/06/07/%E4%BD%BF%E7%94%A8-Vagrant-%E8%B0%83%E8%AF%95-Ansible-Playbook/)。
