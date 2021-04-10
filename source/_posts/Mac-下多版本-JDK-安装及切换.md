---
title: Mac 下多版本 JDK 安装及切换
date: 2021-04-10 19:02:43
categories: Java
tags: Java
---

### 背景
由于公司项目基于 JDK 1.8，我本地默认安装的是 JDK 10，这样在 idea 中通过 maven 编译的时候各种报错，有点不兼容。为了解决这个问题最好的办法就是再安装一个 1.8 的 JDK 版本了，和公司项目代码版本兼容。本文主要介绍 Mac 下如何安装 JDK 并且多版本如何切换。

### Mac 下 JDK 安装配置
Mac 下安装 JDK 比较简单，只需要访问 [Oracle 官网](https://www.oracle.com/index.html)，找到对应环境和版本的 JDK 下载安装即可，下载 mac 下的 dmg 安装文件，一路点击下一步即可。
JDK 下载页面见这里: [https://www.oracle.com/cn/java/technologies/javase-downloads.html](https://www.oracle.com/cn/java/technologies/javase-downloads.html)

mac 下安装 jdk 1.8 的话只需要下载这个安装包即可：jdk-8u281-macosx-x64.dmg

同时安装其他版本，只需要下载其他版本的 dmg 安装包安装即可。

### Mac 下多版本 JDK 切换
1、使用如下命令查看本地安装了哪些 JDK 版本及对应的安装路径：
```bash
haohao@haohaodeMacBook-Pro  ~  /usr/libexec/java_home -V                                                                                                                                                                    ✔  11:03:12
Matching Java Virtual Machines (2):
    10.0.2, x86_64:	"Java SE 10.0.2"	/Library/Java/JavaVirtualMachines/jdk-10.0.2.jdk/Contents/Home
    1.8.0_281, x86_64:	"Java SE 8"	/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home

/Library/Java/JavaVirtualMachines/jdk-10.0.2.jdk/Contents/Home
```
可以看出当前系统有两个 JDK 版本和对应的安装路径。

2、切换到对应的 JDK 版本

设置 `JAVA_HOME` 环境变量为 JDK 安装路径，`vim ~/.zshrc`
```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home
```
source 一下让配置生效: `source ~/.zshrc`

3、检查版本是否切换成功
```bash
haohao@haohaodeMacBook-Pro  ~  java -version                                                                                                                                                                                ✔  11:11:27
java version "1.8.0_281"
Java(TM) SE Runtime Environment (build 1.8.0_281-b09)
Java HotSpot(TM) 64-Bit Server VM (build 25.281-b09, mixed mode)
```

