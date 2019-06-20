---
title: Amazon CloudFront CDN + s3 源站跨域配置
date: 2019-05-06 19:48:25
categories: Aws
tags: Aws
---

### 问题描述
使用 Amazon CloudFront CDN + s3 源站托管前端静态页面，前端跨域请求时报错：
```
...blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.
```
### 解决方法
配置 Amazon CloudFront CDN 和 s3 支持跨域请求
#### 1. s3 存储桶添加 CORS 配置
存储桶--->权限--->CORS配置，添加类似下面 xml 格式的 CORS 配置：
```
<?xml version="1.0" encoding="UTF-8"?>
   <CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
   <CORSRule>
     <AllowedOrigin>*</AllowedOrigin>
     <AllowedMethod>HEAD</AllowedMethod>
     <AllowedMethod>PUT</AllowedMethod>
     <AllowedMethod>GET</AllowedMethod>
     <MaxAgeSeconds>3000</MaxAgeSeconds>
     <ExposeHeader>x-amz-server-side-encryption</ExposeHeader>
     <ExposeHeader>x-amz-request-id</ExposeHeader>
     <ExposeHeader>x-amz-id-2</ExposeHeader>
     <AllowedHeader>*</AllowedHeader>
    </CORSRule>
   </CORSConfiguration>
```
 s3 CORS 相关配置项说明：
 - `<AllowedOrigin>*</AllowedOrigin>`: 允许访问来源，* 表示允许所有来源访问，具体根据实际情况配置；
 - `<AllowedMethod>HEAD</AllowedMethod>`: 允许的请求方法：GET、PUT、POST、DELETE、HEAD，不包含 OPTIONS 请求；
 - `<MaxAgeSeconds>3000</MaxAgeSeconds>`: 指定在 Amazon S3 针对特定资源的预检 OPTIONS 请求做出响应后，浏览器缓存该响应的时间（以秒为单位，在本示例中为 3000 秒）。通过缓存响应，在需要重复原始请求时，浏览器无需向 Amazon S3 发送预检请求。
 - 其他配置项解释见这里：https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/dev/cors.html
      
      
使用 curl 测试存储桶 CORS 配置是否正确：
```
  curl -I -v -L -H 'origin: <跨域请求的来源域名>' <s3资源地址>
```
如果响应头中有如下请求头，则表示配置正确：
```
  Access-Control-Allow-Origin: <curl 请求时 -H 参数指定的值>
  Access-Control-Allow-Methods: <s3 存储桶 CORS 配置指定的请求方法>
```

#### 2. CloudFront 分发行为中配置正确的"白名单标头":
打开 Amazon CloudFront 控制台--->点击要配置的分发--->选中"行为"列--->选中某条行为配置行，点击"编辑"--->"白名单标头"添加如下标头（CORS 相关配置，必须得添加，否则跨域请求时会出问题）：
```
  Access-Control-Request-Headers
  Access-Control-Request-Method
  CloudFront-Forwarded-Proto
  Origin
```
#### 3. CloudFront 缓存行为允许 OPTIONS 请求：
打开 Amazon CloudFront 控制台--->点击要配置的分发--->选中"行为"列--->选中某条行为配置行，点击"编辑"--->"缓存的 HTTP 方法" 下面勾选 OPTIONS 复选框

### 相关资料
   https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/dev/cors.html
   https://aws.amazon.com/cn/premiumsupport/knowledge-center/no-access-control-allow-origin-error/?nc1=h_ls

