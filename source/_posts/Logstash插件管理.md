---
title: Logstash插件管理
date: 2018-01-07 19:58:57
categories: 翻译
tags: 翻译
---

本文翻译自[Elast stack官方文档](https://www.elastic.co/guide/en/logstash/5.5/working-with-plugins.html)，主要介绍Logstash插件的使用方法。

----
#### 译文：
Logstash拥有丰富的input，filter，codec和output插件。插件是独立的gems包，托管在[RubyGems.org](https://rubygems.org)。插件管理器是通过bin/logstash-plugin脚本来使用，用来管理Logstash的插件的生命周期。通过下面的描述，你可以使用命令行来安装，删除，和升级插件。
#### 代理配置:
大部分插件管理命令需要访问[RubyGems.org](https://rubygems.org)。如果你的组织被网络防火墙阻挡，你可以设置环境变量来配置Logstash使用代理。
```bash
export http_proxy=http://localhost:3128
export https_proxy=http://localhost:3128
```
#### 列出插件
Logstash release包已经捆绑了常用的插件，所以你可以直接使用他们。列出当前可用的插件：
```bash
bin/logstash-plugin list 
bin/logstash-plugin list --verbose 
bin/logstash-plugin list '*namefragment*' 
bin/logstash-plugin list --group output 
```
1. 列出所有已安装插件；
2. 列出所有已安装的插件，包括版本信息；
3. 列出所有包括namefragment的已安装插件；
4. 列出指定组的已安装插件(input, filter, codec, output)；

#### 给你的部署环境安装插件
最常见的情况是你通过互联网处理插件安装。使用这个方法，你能够检索托管在[RubyGems.org](https://rubygems.org)的插件并且安装在你的Logstash上。
```bash
bin/logstash-plugin install logstash-output-kafka
```
#### 进阶：安装本地编译好的插件
在某些情况下，你可能需要安装还没有发布且没有托管在[RubyGems.org](https://rubygems.org)的插件。Logstash为您提供了选择，你可以安装本地已编译好的，ruby gem插件包。使用一个文件位置：
```bash
bin/logstash-plugin install /path/to/logstash-output-kafka-1.0.0.gem
```
#### 使用 --path.plugins
使用Logstash --path.plugins选项，你可以加载在你的文件系统上的插件源码。通常情况下这个被开发人员用来迭代自定义的插件并在创建gem包之前做测试。路径需要位于特定的目录层次结构中：PATH/logstash/TYPE/NAME.rb，TYPE是input, outputs或者codes NAME是插件的名字。
```bash
# supposing the code is in /opt/shared/lib/logstash/inputs/my-custom-plugin-code.rb
bin/logstash --path.plugins /opt/shared/lib
```
#### 升级插件
插件有自己的发布周期，通常独立于Logstash的发布周期。使用update子命令，你可以获取到最新版本的插件。
```bash
bin/logstash-plugin update 
bin/logstash-plugin update logstash-output-kafka
```
1. 更新所有已安装插件；
2. 只更新指定的插件；

#### 删除插件
如果你需要从Logstash的安装中删除插件：
```bash
bin/logstash-plugin remove logstash-output-kafka
```
#### 代理支持
之前的章节依赖于Logstash能够访问[RubyGems.org](https://rubygems.org)。在某些环境下，正向代理用来处理HTTP请求。可以通过设置HTTP_PROXY环境变量来安装和更新Logstash插件：
```bash
export HTTP_PROXY=http://127.0.0.1:3128
bin/logstash-plugin install logstash-output-kafka
```
一旦设置了后，就可以使用这个代理来进行插件的安装和更新。
