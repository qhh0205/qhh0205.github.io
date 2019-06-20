---
title: GitHub设置fork仓库和原始仓库同步
date: 2018-02-16 13:53:13
categories: Git
tags: GitHub
---

#### 问题描述
最近fork了一个翻译项目[Linux中国翻译项目(LCTT)](https://github.com/LCTT/TranslateProject)，准备用自己的业余时间为社区贡献点自己的力量，发现这个原始仓库比较活跃，经常出现fork仓库比原始仓库落后的情况：
![](/images/github-sync1.png)
可以看出该仓库已经落后30个提交了，因此，为了避免长时间不同步原始仓库导致后面的PR可能发生冲突等其它问题，需要手动定期同步下fork仓库。
#### 配置fork仓库和原始(上游)仓库同步
下面配置该[fork仓库](https://github.com/qianghaohao/TranslateProject)和[原始仓库](https://github.com/LCTT/TranslateProject)的同步：
1.本地fork仓库配置upstream地址
默认情况下clone的仓库只有两个远程地址，用来fetch和push时使用：
```
$ git remote -v
origin	github:qianghaohao/TranslateProject.git (fetch)
origin	github:qianghaohao/TranslateProject.git (push)```

为了和原始(上游)仓库同步，我们还需要为该仓库配一个upstream地址，用来同步时使用，配置方法如下：
```bash
# git remote add upstream后面添加要同步的原始仓库地址
git remote add upstream git@github.com:LCTT/TranslateProject.git
# 重新查看远程仓库配置，发现upstream已经配置成功
git remote -v
origin	github:qianghaohao/TranslateProject.git (fetch)
origin	github:qianghaohao/TranslateProject.git (push)
upstream	git@github.com:LCTT/TranslateProject.git (fetch)
upstream	git@github.com:LCTT/TranslateProject.git (push)
```
2.从上游分支fetch最新更改
本示例仓库只有一个master分支，所以在这里只同步下master分支。fetch后会将该分支的上游更新存储到新的本地分支upstream/master：
```bash
# 可以看出fetch后在本地生成了新的本地分支：upstream/master
$ git fetch upstream
remote: Counting objects: 139, done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 139 (delta 73), reused 73 (delta 73), pack-reused 62
Receiving objects: 100% (139/139), 58.17 KiB | 5.00 KiB/s, done.
Resolving deltas: 100% (73/73), completed with 17 local objects.
From github.com:LCTT/TranslateProject
 * [new branch]      master     -> upstream/master
```
3.合并上一步本地fetch得到的分支到本地相应分支
```bash
$ git merge upstream/master
Updating db2c3cc..0175778
Fast-forward
 published/20171016 Using the Linux find command with caution.md                             |  97 ++++++++++++++++++
 published/20171117 How to Install and Use Docker on Linux.md                                | 165 +++++++++++++++++++++++++++++++
 published/20171228 Dual Boot Ubuntu And Arch Linux.md                                       | 422 ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 published/20180118 Getting Started with ncurses.md                                          | 197 ++++++++++++++++++++++++++++++++++++
 published/20180202 Which Linux Kernel Version Is Stable.md                                  |  59 +++++++++++
 {translated/tech => published}/20180206 Save Some Battery On Our Linux Machines With TLP.md |  24 ++---
 sources/talk/20180214 How to Encrypt Files with Tomb on Ubuntu 16.04 LTS.md                 |   4 +-
```
4.push合并后的修改到[远程fork仓库](https://github.com/qianghaohao/TranslateProject)，完成同步
```bash
$ git push
Counting objects: 139, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (77/77), done.
Writing objects: 100% (139/139), 51.50 KiB | 0 bytes/s, done.
Total 139 (delta 69), reused 126 (delta 62)
remote: Resolving deltas: 100% (69/69), completed with 12 local objects.
To github:qianghaohao/TranslateProject.git
   db2c3cc..0175778  master -> master
```
可以看出[远程fork仓库](https://github.com/qianghaohao/TranslateProject)已经是最新的，和[原始仓库](https://github.com/LCTT/TranslateProject)保持一致：
![](/images/github-sync2.png)
#### 参考文章
[https://help.github.com/articles/syncing-a-fork/](https://help.github.com/articles/syncing-a-fork/)
[http://wiki.jikexueyuan.com/project/github-basics/fork-synced.html](http://wiki.jikexueyuan.com/project/github-basics/fork-synced.html)

