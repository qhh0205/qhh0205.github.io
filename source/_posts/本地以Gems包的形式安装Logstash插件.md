---
title: 本地以Gems包的形式安装Logstash插件
date: 2018-01-13 20:56:20
categories: ELK
tags: Logstash
---

#### 概述
Logstash的插件都是独立的gem包，因此可以通过从[RubyGems.org](https://rubygems.org/)来下载需要的插件的gem包来安装Logstash插件。[RubyGems.org](https://rubygems.org/)是一个专门用来托管gem包的网站，类似于yum包的仓库，上面存放各种Ruby gem包供用户下载并使用。
#### 安装过程
 以下通过安装最近刚发布的logstash-filter-dissect v1.1.1插件包为例来说明安装过程，logstash-filter-dissect v1.1.1修复了我提的这个[issue#41](https://github.com/logstash-plugins/logstash-filter-dissect/issues/41)及其他一些bug，具体请看[CHANGELOG](https://github.com/logstash-plugins/logstash-filter-dissect/blob/v1.1.1/CHANGELOG.md)。
 1. 打开[RubyGems.org](https://rubygems.org/)官网，找到我们需要的logstash-filter-dissect v1.1.1 gem包并下载，下载下来是gem文件:
![](/images/gem-org1.png)
![](/images/gem-org2.png)
![](/images/gem-org3.png)

 2. 使用bin/logstash-plugin install命令来安装下载的gem包:
```bash
# 删除此插件的当前版本
bin/logstash-plugin remove logstash-filter-dissect
# 安装下载的gem包
bin/logstash-plugin install ../logstash-filter-dissect-1.1.1.gem
#查看版本是否是安装的版本
bin/logstash-plugin list --verbose | grep dissect
# logstash-filter-dissect (1.1.1)  可以看到已经安装成功
```
