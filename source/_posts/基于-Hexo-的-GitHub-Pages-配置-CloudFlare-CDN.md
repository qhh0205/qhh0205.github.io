---
title: 基于 Hexo 的 GitHub Pages 配置 CloudFlare CDN
date: 2018-11-04 11:53:00
categories: Hexo
tags: Hexo
---

### 概述
由于 GitHub Pages 在国外，静态博客页面在国内访问速度可能会非常慢，我们可以用 CDN 来加速，对比了下 CloudFlare CDN 和 腾讯云 CDN，发现 CloudFlare 免费版没有流量限制（腾讯云 CDN 每月由流量限制），而且配置起来非常简单，所以在此选用 CloudFlare CDN 来加速页面访问。

### 准备工作
* 个人域名
* CloudFlare 账号
* 基于 hexo 的 github_username.github.io 静态博客

### 配置流程
1. 在 Hexo 博客 source 文件夹新建名为 CNAME 的文件，内容为个人域名；
2. `hexo g && hexo d` 部署生产的静态页面到 GitHub；
3. 进入 CloudFlare 控制台，点击添加站点，输入个人域名，根据向导进行操作；
4. 在 CloudFlare DNS 配置页面配置两个 CNAME 均指向 github_username.github.io 地址：
根域名(@) CNAME 到 `github_username.github.io`
子域名(www) CNAME 到 `github_username.github.io`
> ⚠️注意：其实一般的域名提供商是不支持根域名 CNAME ，只有子域名才可以，但是 CloudFlare 通过 [CNAME Flattening](https://support.cloudflare.com/hc/en-us/articles/200169056-CNAME-Flattening-RFC-compliant-support-for-CNAME-at-the-root) 技术支持这种配置。这么做的好处是我们不需要再一个个添加以 GitHub Pages 的 IP 为值的 A 记录了，同时还能提高后续的可维护性，后续即使 GitHub Pages 的 IP 发生了变化，也不影响，CloudFlare 会通过 [CNAME Flattening](https://support.cloudflare.com/hc/en-us/articles/200169056-CNAME-Flattening-RFC-compliant-support-for-CNAME-at-the-root) 技术 自动解析出来。
> 

	![](/images/cloudflare1.png)
5. 将个人域名的 NS 记录修改为 CloudFlare 的 NS；
6. 等 CloudFlare DNS 解析生效后，并且 CloudFlare 站点状态为 Active 即表示配置生效。
![](/images/cloudflare2.png)

### CloudFlare CDN HTTP 强制跳转 HTTPS
默认情况下配置完成后 HTTPS 是开启的，会在 24 小时内给你配的站点颁发 https 证书，并且证书是自动更新的。我们可以在 CloudFlare 控制台配置 HTTP 强制跳转 HTTPS：
![](/images/cloudflare3.png)
![](/images/cloudflare4.png)
