---
title: Nginx upstream 失效转移机制研究
date: 2018-07-08 14:15:40
categories: Nginx
tags: Nginx
---

### 结论
经过多次模拟线上的环境测试，Nginx 负载均衡技术默认情况下已经对于 `connect refused`（状态码表现为 `502`）和 `time out`（状态码表现为 `504`）做了失效转移，使用的是 `upstream` 模块的 `proxy_next_upstream` 指令（这个选项默认是启动的）来实现。

对于 `http GET` 请求，当这个请求转发到上游服务器发生断路，或者读取响应超时则会将同样的请求转发到其他上游服务器来处理，如果所有服务器都超时或者断路，则会返回 `502` 或者 `504` 错误。

对于` http POST` 请求，当这个请求转发到上游服务器发生断路，则会将请求转发到其他上游服务器来处理，但是如果这个请求发生了读取超时，则不会做失效转移，会返回 `504` 错误，Nginx 之所以这么做应该是为了防止同一个请求发送两次，比如涉及到银行的充值等操作就会发生很严重的 bug。以下是模拟线上的场景测试得出的结论：

1. 上游服务器有两台，一台处于 down 状态，另一台处于正常服务状态，那么来自客户端的 `GET` 和 `POST` 请求都会通过 Nginx 的失效转移机制路由到正常状态的机器，返回 `200` 状态码，并不会返回给客户端 `502` 错误；
2. 上游服务器有两台，两台都 down 了，那么会不管是 `GET` 还是 `POST` 请求都会直接返回给客户端 `502` 错误；
3. 上游服务器有两台，一台机器的 `http GET` 和 `POST` 接口都正常 return，另一台相同的接口死循环，模拟超时。
这种情况下如果客户端的请求路由到了正常机器，那么直接返回 `200`。
如果请求路由到了死循环的接口，并且是 `GET` 请求，那么会等待 Nginx 设置的超时时间过后，然后将请求转发到另一台机器的正常接口。
如果请求路由到了死循环的接口，并且是 `POST` 请求，那么等待 nginx 设置的超时时间过后直接返回 `504`，没有进行失效转移，防止请求的重复发送；
4. 上游服务器有两台，两台机器的 `http GET` 和 `POST` 接口都死循环，模拟超时，那么对于  `GET` 请求会进行请求转发到另一台尝试，对于 `POST` 请求直接返回 `504`，不会进行进一步尝试；

### 论证环境及工具
- 一台前端 Nginx 服务器；
- 两台上游服务器；
- Nginx 配置：

```
server {
  listen       443;
  server_name ngxfailover.xxx.me;
  ssl on;
  ssl_certificate xxx/xxx.crt;
  ssl_certificate_key xxx/xxx.key;
  location / {
     proxy_pass http://py_web_upstream;
  }
}

upstream py_web_upstream{
      server upstream_server1:5000;
      server upstream_server2:5000;
}
```
- 第一台上游服务器正常代码

```python
import time
from flask import Flask
app = Flask(__name__)


@app.route('/a/<name>')
def failover_get_method(name):
    print name
    return '<h1>I am Server2, My name is %s </h1>' % name

@app.route('/b/<name>', methods=["POST"])
def failover_post_method(name):
    print name
    return '<h1>I am Server2, My name is %s </h1>' % name


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```
- 第二台上游服务器超时代码

```python
import time
from flask import Flask
app = Flask(__name__)


@app.route('/a/<name>')
def failover_get_method(name):
    print name
    while True:
        time.sleep(256)
    return '<h1>I am Server2, My name is %s </h1>' % name

@app.route('/b/<name>', methods=["POST"])
def failover_post_method(name):
    print name
    while True:
        time.sleep(256)
    return '<h1>I am Server2, my name is %s </h1>' % name


if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

### 论证过程
以下为前面 4 种案例的论证过程：

#### 案例 1
上游服务器有两台，一台处于 down 状态，另一台处于正常服务状态。

在这种情况下，通过 `curl` 多次发送 `GET` 和 `POST` 请求，发现不管怎么请求，返回都是正常状态，如果 Nginx 发生了失败尝试操作，那么会在 Nginx access 日志中的 upstream 字段看到有两个服务器的地址。

**发送 GET 和 POST 请求：**

```bash
curl -XGET https://ngxfailover.xxx.me/a/hello
curl -XPOST https://ngxfailover.xxx.me/b/hello
```
**观察日志：**

可以看出所有请求都成功了，红框框圈起来的请求表示发生了失效转移，并且请求成功。
![](/images/ngx_failover_log1.png)
#### 案例 2
上游服务器有两台，两台服务器都处于 down 状态。

在这种情况下不管是 `GET` 还是 `POST` 请求都会直接返回给客户端 `502` 错误。

**发送 GET 和 POST 请求：**

```bash
curl -XGET https://ngxfailover.xxx.me/a/hello
curl -XPOST https://ngxfailover.xxx.me/b/hello
```

**观察日志：**
可以看出所有请求全部返回 `502` 错误，红框框圈起来的请求表示发生了失效转移，但是还是失败了。
![](/images/ngx_failover_log2.png)
#### 案例 3
上游服务器有两台，一台机器的 `http GET` 和 `POST` 接口都正常 return，另一台相同的接口死循环，模拟超时。

这种情况下如果客户端的请求路由到了正常机器，那么直接返回 `200`。

如果请求路由到了死循环的接口，并且是 `GET` 请求，那么会等待 Nginx 设置的超时时间过后，然后将请求转发到另一台机器的正常接口。

如果请求路由到了死循环的接口，并且是 `POST` 请求，那么等待 Nginx 设置的超时时间过后直接返回客户端 `504` 错误，没有进行失效转移，防止请求的重复发送。

**发送 GET 请求：**

```bash
curl -XGET https://ngxfailover.xxx.me/a/hello
```

**观察日志：**

可以看到对于 `GET` 请求全部成功，红框框圈起来的表示发生了失效转移，第一台超时后会是继续尝试第二台，最终成功。
![](/images/ngx_failover_log3.png)

**发送 POST 请求：**

```bash
curl -XPOST https://ngxfailover.xxx.me/b/hello
```

**观察日志：**

可以看到对于 `POST` 请求，如果 Nginx 等待上游服务器处理请求超时，并不会发生失效转移，直接返回给客户端 `504` 错误。
![](/images/ngx_failover_log4.png)

#### 案例 4
上游服务器有两台，两台机器的 `http GET` 和 `POST` 接口都死循环，模拟超时。

这种情况下对于 `GET` 请求会将请求转发到另一台尝试，对于 `POST` 请求直接返回 `504` 错误，不会进行进一步尝试。

**发送 GET 请求：**

```bash
curl -XGET https://ngxfailover.xxx.me/a/hello
```

**观察日志：**

可以看出对于 `GET` 请求，Nginx 在等待超时会继续进行尝试，两台都尝试失败后返回了 `504` 错误。
![](/images/ngx_failover_log5.png)

**发送 POST 请求：**

```bash
curl -XPOST https://ngxfailover.xxx.me/b/hello
```

**观察日志：**

可以看出对于 `POST` 请求，Nginx 在等待超时会不继续进行尝试其他上游服务器，直接返回 `504` 错误。
![](/images/ngx_failover_log6.png)

### 总结
总体来看 Nginx 的失效转移技术已经非常成熟，Nginx 默认情况下对于 `connect refused`（状态码表现为 `502`）和 `time out`（状态码表现为 `504`）已经做了失效转移，并且 Nginx 根据请求的类型不同，对失效转移的策略也不同。对于服务器后台状态没有改变的请求（比如 `GET` 请求）会进行失效转移，对于服务后台状态有改变的请求（比如 `POST` 请求），有失效转移机制，这也符合 Rest API 的冪等性标准。如果要强行加其他状态码的失效转移，比如 `500`、`503` 等，需要考量下业务请求是否能容忍请求的重复发送。

