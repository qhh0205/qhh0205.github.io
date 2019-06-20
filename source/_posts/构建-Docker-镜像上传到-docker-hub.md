---
title: 构建 Docker 镜像上传到 docker hub
date: 2019-01-13 20:58:16
categories: Docker
tags: Docker
---

本文演示如何将自己构建的 Docker 镜像推送到 [docker hub](https://hub.docker.com/)来实现镜像的共享。

1. 注册一个 docker hub 账号
举例：账号名为 qhh0205
2. 写一个 Dockerfile
举例：该 Dockerfile 安装了指定版本的 ant 和 jmeter，GitHub 仓库地址：https://github.com/qhh0205/docker-ant-jmeter
```
FROM openjdk:8
MAINTAINER qhh0205 <qhh0205@gmail.com>

# ant default version: 1.10.5
# jmeter default version: 5.0
# Specify version by docker build --build-arg <varname>=<value> ...

ARG ANT_VERSION=1.10.5
ENV ANT_HOME=/opt/ant

ARG JMETER_VERSION=5.0
ENV JMETER_HOME /opt/jmeter

RUN apt-get -y update && \
apt-get -y install wget

# Installs Ant
RUN wget --no-check-certificate --no-cookies http://archive.apache.org/dist//ant/binaries/apache-ant-${ANT_VERSION}-bin.tar.gz \
&& tar -zvxf apache-ant-${ANT_VERSION}-bin.tar.gz -C /opt/ \
&& ln -s /opt/apache-ant-${ANT_VERSION} /opt/ant \
&& rm -f apache-ant-${ANT_VERSION}-bin.tar.gz

ENV PATH ${PATH}:/opt/ant/bin

# Installs Jmeter
RUN wget --no-check-certificate --no-cookies https://archive.apache.org/dist//jmeter/binaries/apache-jmeter-${JMETER_VERSION}.tgz \
&& tar -zvxf apache-jmeter-${JMETER_VERSION}.tgz -C /opt/ \
&& ln -s /opt/apache-jmeter-${JMETER_VERSION} /opt/jmeter \
&& rm -f apache-jmeter-${JMETER_VERSION}.tgz

ENV PATH $PATH:/opt/jmeter/bin
```

3. 构建镜像
进入 Dockerfile 所在目录，执行构建命令:
```
docker build -t qhh0205/ant-jmeter:1.10.5-5.0 .
```


	参数说明：
	`qhh0205/ant-jmeter:1.10.5-5.0`: docker 镜像 tag 名称
	`qhh0205`: docker hub 账号名
	`ant-jmeter`: dcoker hub 仓库名
	`1.10.5-5.0`: 镜像 tag

4. 登陆 docker hub 账号
```
docker login
```

5. 上传镜像
```
docker push qhh0205/ant-jmeter:1.10.5-5.0
```

