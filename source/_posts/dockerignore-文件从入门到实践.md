---
title: .dockerignore 文件从入门到实践
date: 2019-02-24 14:13:51
categories: Docker
tags: Docker
---

### 简介
`.dockerignore` 文件的作用类似于 git 工程中的 `.gitignore` 。不同的是 `.dockerignore` 应用于 docker 镜像的构建，它存在于 docker 构建上下文的根目录，用来排除不需要上传到 docker 服务端的文件或目录。

docker 在构建镜像时首先从构建上下文找有没有 `.dockerignore` 文件，如果有的话则在上传上下文到 docker 服务端时忽略掉 `.dockerignore` 里面的文件列表。这么做显然带来的好处是：
- 构建镜像时能避免不需要的大文件上传到服务端，从而拖慢构建的速度、网络带宽的消耗；
- 可以避免构建镜像时将一些敏感文件及其他不需要的文件打包到镜像中，从而提高镜像的安全性；

### .dockerignore 文件编写方法
`.dockerignore` 文件的写法和 `.gitignore` 类似，支持正则和通配符，具体规则如下：
- 每行为一个条目；
- 以 `#` 开头的行为注释；
- 空行被忽略；
- 构建上下文路径为所有文件的根路径；

**文件匹配规则具体语法如下：**

| 规则 | 行为 |
| :----- | :----- |
| \*/temp\* | 匹配根路径下一级目录下所有以 temp 开头的文件或目录 |
| \*/\*/temp\* | 匹配根路径下两级目录下所有以 temp 开头的文件或目录 |
| temp? | 匹配根路径下以 temp 开头，任意一个字符结尾的文件或目录 |
| \*\*/\*.go | 匹配所有路径下以 `.go` 结尾的文件或目录，即递归搜索所有路径 |
|*.md<br>!README.md|匹配根路径下所有以 `.md` 结尾的文件或目录，但 README.md 除外|

⚠️注意事项：
如果两个匹配语法规则有包含或者重叠关系，那么以后面的匹配规则为准，比如：
```
*.md
!README*.md
README-secret.md
```
这么写的意思是将根路径下所有以 `.md` 结尾的文件排除，以 `README` 开头 `.md` 结尾的文件保留，但是 `README-secret.md` 文件排除。

再来看看下面这种写法（同上面那种写法只是对换了后面两行的位置）：
```
*.md
README-secret.md
!README*.md
```
这么写的意思是将根路径下所有以 `.md` 结尾和名称为 `README-secret.md` 的文件排除，但所有以 `README` 开头 `.md` 结尾的文件保留。这样的话 `README-secret.md` 依旧会被保留，并不会被排除，因为 `README-secret.md` 符合 `!README*.md` 规则。

### 使用案例
前段时间帮前端同学写了一个 Dockerfile，Dockerfile 放在 git 仓库根路径下，发现 git 工程中有很多真正应用跑起来用不到的文件，如果直接在 Dockerfile 中使用 `COPY` 或 `ADD` 指令拷贝文件，那么很显然会把很多不需要的文件拷贝到镜像中，从而会拖慢构建镜像的过程，产生的镜像也比较臃肿。解决方法就是编写 `.dockerignore` 文件，忽略掉不需要的文件，然后放到 docker 构建上下文的根路径下。`.dockerignore` 及 `Dockerfile` 文件内容如下：
.dockerignore:
```
.git
_mockData
deleted
email-templates
script
static
```
Dockerfile:
```
FROM node:8-alpine

COPY . /app/node
WORKDIR /app/node
RUN yarn install

EXPOSE 8026

CMD ["yarn", "run", "tool-dev"]
```
使用 `.dockerignore` 前后上传到 docker 服务端的构建上下文大小对比：
使用前（`73.36MB`）：
```
[vagrant@docker]$ docker build -t tool:5.0 -f Dockerfile-frontend-tool .
Sending build context to Docker daemon  73.36MB
Step 1/6 : FROM node:8-alpine
```
使用后（`11.38MB`）：
```
[vagrant@docker]$ docker build -t tool:6.0 -f Dockerfile-frontend-tool .
Sending build context to Docker daemon  11.38MB
Step 1/6 : FROM node:8-alpine
```

### 参考资料
https://docs.docker.com/engine/reference/builder/#dockerignore-file
