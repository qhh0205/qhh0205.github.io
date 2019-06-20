---
title: 使用LFS解决GitHub无法上传大文件问题
date: 2018-01-20 22:22:30
categories: Git
tags: GitHub
---

今天使用GitHub上传几个比较大的pdf电子书，有的大小超过100MB了，结果GitHub报错提示无法上传大于100MB的文件，报错信息如下：
```bash
remote: warning: File pdf/深入理解Java虚拟机:JVM高级特性与最佳实践.pdf is 61.47 MB; this is larger than GitHub's recommended maximum file size of 50.00 MB        
remote: error: GH001: Large files detected. You may want to try Git Large File Storage - https://git-lfs.github.com.        
remote: error: Trace: e4e7c6c5ccdc4241f8d9bb334fd46ba8        
remote: error: See http://git.io/iEPt8g for more information.        
remote: error: File pdf/Nginx高性能Web服务器详解.pdf is 183.19 MB; this exceeds GitHub's file size limit of 100.00 MB 
```
仔细看了下报错信息，发下可以使用GitHub的[LFS(Large File Storage)服务](https://git-lfs.github.com)来实现上传大文件。
#### GitHub LFS简介：
GitHub LFS是一个开源的git扩展，可以让git追踪大文件的版本信息。LFS使用文件指针来代替大文件，如音频文件，视频文件，数据采集和图形等文件，同时将文件内容存储到远程服务器，比如GitHub.com或者GitHub Enterprise。LFS是GitHu所支持的一种完全免费的服务，目的是让git能跟踪大文件。
![LFS](/images/github-lfs.png)
#### 如何使用GitHub LFS让git处理大文件
1. 安装
如果是mac系统则直接执行如下命令安装:
```
brew install git-lfs
```

2. 进入本地仓库目录初始化LFS
```
git lfs install
```

3. 用git lfs管理大文件
用git lfs track命令跟踪特定后缀的大文件，或者也可以直接编辑.gitattributes，类似与.gitignore文件的编写，在此我只处理pdf文件：
```bash
git lfs track "*.pdf"
git add .gitattributes
```

4. 接下来就可以像平时使用git那样正常使用了，可以将大文件提交到GitHub了
```
git add . 
git commit -m "update"
git push origin hexo
```

#### 参考文档
[Git Large File Storage | https://git-lfs.github.com](https://git-lfs.github.com/)
