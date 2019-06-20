---
title: Python 根据本地时间获取 UTC 偏移量
date: 2018-06-09 12:50:09
categories: Python
tags: Python
---

在 [so](https://stackoverflow.com/) 上查了一下，可以用如下代码获取本地时间相对于 UTC 时间的偏移量，代码实现思路比较简单，分别获取本地时间和和 UTC 时间，然后本地时间减去 UTC 时间即可得到相对于 UTC 的偏移小时，代码如下：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# @Time    : 2018/4/6 上午10:28
# @Author  : qhh0205
# @Mail    : qhh0205@gmail.com
# @File    : utc_offset.py

import time
from datetime import datetime
ts = time.time()
utc_offset = int((datetime.fromtimestamp(ts) - datetime.utcfromtimestamp(ts)).total_seconds() / 3600)
print "UTC%+-d" % utc_offset

```
参考链接（so 讨论）：
https://stackoverflow.com/questions/3168096/getting-computers-utc-offset-in-python?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa

