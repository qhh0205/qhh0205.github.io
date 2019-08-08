---
title: Mac OS 启用 ssh 远程登陆
date: 2019-08-08 18:49:59
categories: Mac
tags: Mac
---

检查 ssh 远程登陆是否启用
```bash
sudo systemsetup -getremotelogin
```
启用 ssh 远程登陆
```
sudo systemsetup -setremotelogin on
```
启用后就可以用 ssh 来登陆 mac 系统了，账号和密码为系统的账号密码。

关闭 ssh 远程登陆
```
sudo systemsetup -f -setremotelogin off
```
