---
title: Markdown 页面内跳转
date: 2018-03-18 09:21:00
categories: Markdown
tags: Markdown
---

#### Markdonwn 页面内跳转
##### 问题描述
有时候在 Markdown 中需要点击某个链接，然后跳转到当前页面内的其他地方，而不是跳转到一个URL链接，这个在 Markdown 是可以实现的。
##### 解决问题

1.使用 html 标签定义一个锚（id），将需要跳转的内容使用该标签包起来:
```
# id 可以随便定义，只要同一页面内保证唯一即可
<span id="1">
要跳转的内容（在这里可以写任意内容，MD 格式的也可以）
</span>
```

2.后在页面内超链接到指定锚（id）上：
```
[链接文本](#1)
```
举例：
[~/.ssh/config GitHub Host配置](#1)
点击上面链接即可跳转到该页面中链接的代码块。
<span id=1>
```
Host github
      HostName github.com
      User git
      IdentityFile ~/.ssh/my_github_key
```
</span>
