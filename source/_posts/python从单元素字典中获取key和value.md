---
title: Python从单元素字典中获取key和value
date: 2017-12-03 22:37:17
categories: Python
tags: Python
---
之前写代码很多时候会遇到这么一种情况:在python的字典中只有一个key/value键值对，想要获取其中的这一个元素还要写个for循环获取。网上搜了一下，发现还有很多简单的方法:
* 方法一
``` python
d = {'name':'haohao'}
(key, value), = d.items()
```
* 方法二
``` python
d = {'name':'haohao'}
key = list(d)[0]
value = list(d.values())[0]
```
* 方法三
``` python
d = {'name':'haohao'}
key, = d
value, = d.values()
```
参考自stackoverflow讨论:
[https://stackoverflow.com/questions/15366482/how-to-fetch-the-key-value-pair-of-a-dictionary-only-containing-one-item](https://stackoverflow.com/questions/15366482/how-to-fetch-the-key-value-pair-of-a-dictionary-only-containing-one-item)

