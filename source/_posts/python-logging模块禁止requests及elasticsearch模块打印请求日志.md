---
title: python logging模块禁止requests及elasticsearch模块打印请求日志
date: 2018-02-18 18:21:41
categories: Python
tags: Python
---

最近写的代码基本都用到了python的标准日志模块logging，但发现在使用requests模块和elasticsearch时，即使自己没有打印相关日志，也会自动生成请求过程日志，示例如下：

 * **requests日志**
```
 2017-11-02 17:30:31|INFO|Starting new HTTP connection (1): elk
 2017-11-03 06:32:55|INFO|Starting new HTTP connection (1): elk
```
 
* **elasticsearch日志**
```
2017-09-25 02:00:01|INFO|GET http://localhost:9200/configcenter*/_search [status:200 request:0.020s]
2017-09-25 02:00:01|INFO|GET http://localhost:9200/abc*/_search [status:200 request:0.007s]
2017-09-25 02:00:01|INFO|GET http://localhost:9200/def*/_search [status:200 request:0.008s]
```

上面这种日志我们是不需要的，如果这种日志和我们自己打的日志混合在一块儿，日志文件将变得难以查看，对后面的问题排查带来很多不便，因此我们需要禁用掉这种默认的日志打印，方法如下：

* **requests模块请求日志禁用:**
```python
logging.getLogger("requests").setLevel(logging.WARNING)
```
* **elasticsearch模块请求日志禁用:**
```python
logging.getLogger("elasticsearch").setLevel(logging.WARNING)
```
**参考文章:** 
[https://stackoverflow.com/questions/11029717/how-do-i-disable-log-messages-from-the-requests-library](https://stackoverflow.com/questions/11029717/how-do-i-disable-log-messages-from-the-requests-library)
