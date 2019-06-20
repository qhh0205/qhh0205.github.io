---
title: GitHub多账号密钥配置
date: 2018-02-19 16:56:46
categories: Git
tags: GitHub
---

#### GitHub多账号密钥配置
#### 问题描述
之前在自己的GitHub账号配置过本地电脑的ssh key，然后用同样的ssh key给另外一个GitHub账号配置密钥，发现报如下错误：
![](/images/github1.jpg)
其实这个问题的原因很简单，主要是GitHub账号和服务器通信的ssh密钥对是一一对应的，不允许多个账号使用同样的密钥对，否则会出现歧义，需要为每个账号配置不同的密钥对。
#### 解决方法
1. 为每个GitHub账号单独生成ssh密钥对：
```bash
ssh-keygen -t rsa -f github1 -C "abc@gmail.com"
ssh-keygen -t rsa -f github2 -C "def@gmail.com"
```
2. 把生成的公钥分别添加到GitHub账号管理后台：
![](/images/github2.png)
3. 编辑~/.ssh/config文件添加如下内容：
```bash
# 其中Host是主机别名，HostName是github服务器地址，User是GitHub服务器用户名，
# IdentityFile是和GitHub服务器通信的ssh私钥，通过IdentityFile就可以区分出
# 不同的账号。
Host account1
    HostName github.com
    User git
    IdentityFile ~/.ssh/github1
Host account2
    HostName github.com
    User git
    IdentityFile ~/.ssh/github2
```
4. 使用ssh-agent管理生成的私钥：
```bash
ssh-add github1
ssh-add github2
```
5. 在使用git clone时将GitHub SSH仓库地址中的git@github.com替换成第三步新建的Host别名account1或account2（仓库属于哪个Host则使用哪个，这里假设仓库属于account1，GitHub账号的区分是通过在GitHub管理后台添加的公钥来辨识）。如原地址是：*git@github.com:qianghaohao/TranslateProject.git* 替换后应该是：*account1:qianghaohao/TranslateProject.git* 如果是新建的仓库，直接使用替换后的URL克隆即可。如果已经使用原地址克隆过了，可以使用命令修改远程仓库地址：
```bash
git remote set-url origin account1:qianghaohao/TranslateProject.git
```
#### 参考文章
[http://blog.csdn.net/u010387196/article/details/41266255](http://blog.csdn.net/u010387196/article/details/41266255)
[http://happy123.me/blog/2014/12/07/duo-ge-gitzhang-hao-zhi-jian-de-qie-huan](http://happy123.me/blog/2014/12/07/duo-ge-gitzhang-hao-zhi-jian-de-qie-huan/)
