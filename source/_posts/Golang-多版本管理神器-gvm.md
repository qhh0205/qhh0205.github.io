---
title: Golang 多版本管理神器 gvm
date: 2020-03-01 11:59:31
categories: Go
tags: Go
---

### 缘起
最近编译 kubernetes 遇到了点坑，编译各种报错，经搜索调研发现 k8s 的编译对 go 的版本有很严格的要求。比如我的 go1.13.4 就无法编译 kubernetes v1.16.3，必须得 go1.12.xx 版本才能编译。为了解决这种尴尬的场景只能再在主机安装个 go1.12.xx 版本，那么有没有什么优雅的方式来实现本机多版本 Golang 版本的管理呢，能很方便的进行不同版本的切换，这也是本文的目的，推荐一款 Go多版本管理神器 [gvm](https://github.com/moovweb/gvm)，用法类似 Python 的多版本管理工具 [pyenv](https://github.com/pyenv/pyenv)。

### 简介
[gvm](https://github.com/moovweb/gvm)，即 Go Version Manager，Go 版本管理器，使用 shell 脚本开发，它可以非常轻量的切换 Go 版本。对比其他语言，通常也有类似的工具，如 NodeJS 的 NVM，Python 的 pyenv 等。在使用方法上和 Python 的多版本管理工具 pyenv 非常类似。

其实不借助类似的版本管理工具安装多个版本 Go 也是可以自己手动实现的，做法很简单，就是下载不同的 Golang 安装包，然后放置到独立的目录，使用时将 GOROOT 和 GOPATH 指向对应版本的目录即可完成版本切换。其实 gvm 原理上就是这么做的，只不过通过工具的形式将这些繁杂的手工操作封装起来，使得开发起来更加优雅，不必再为 Go 的安装、版本管理花费更多的心思。下面为 gvm 的工作原理：

![gvm](/images/gvm.png)

### 安装
```bash
bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```
安装可能遇到的坑:
```bash
$ gvm  install go1.12.10
Downloading Go source...
ERROR: Couldn't download Go source. Check the logs /Users/jim/.gvm/logs/go-download.log
```
根据提示看 log 报错
```bash
Cloning into '/Users/jim/.gvm/archive/go'...
error: RPC failed; curl 18 transfer closed with outstanding read data remaining
fatal: the remote end hung up unexpectedly
fatal: early EOF
fatal: index-pack failed
```

解决问题：`vim ~/.gvm/scripts/install`
修改 GO_SOURCE_URL 变量地址为: GO_SOURCE_URL=git://github.com/golang/go

### 使用速记
1.列出当前已安装的 Go 版本
```bash
gvm list
```
2.列出当前可以安装的 Go 版本
```bash
gvm listall
```
3.安装指定版本的 Go
```bash
gvm install go1.12.10
```
4.切换到指定的 Go 版本
临时切换
```bash
gvm use go1.12
```
永久切换
```bash
gvm use go1.12 --default
```

### 参考资料
[https://juejin.im/post/5d848b66f265da03a7160e89](https://juejin.im/post/5d848b66f265da03a7160e89)
