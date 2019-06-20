---
title: 在字符串列表中找出与s最长前缀匹配的字符串
date: 2017-12-03 11:58:42
categories: Python
tags: Python
---

### 在字符串列表中找出与s最长前缀匹配的字符串
``` python
def closest_match(s, str_list):
    """
    在字符串列表中找出与s最长前缀匹配的字符串
    :param s:
    :param str_list:
    :return: 如果没有任何匹配则返回空串，否则返回最长前缀匹配
    """
    closest = ""
    for str in str_list:
        if s.startswith(str):
            if closest:
                if len(str) >= len(closest):
                    closest = str
            else:
                closest = str
    return closest
```
