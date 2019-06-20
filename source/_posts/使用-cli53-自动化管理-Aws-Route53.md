---
title: 使用 cli53 自动化管理 Aws Route53
date: 2018-06-10 09:18:44
categories: Aws
tags: AWS
---

### cli53 工具
cli53 是一个开源的命令行管理 Aws Route53 的工具，非常实用，可以通过命令行来进行域名及相关记录的创建、更新、删除、记录的导出备份、记录的导入恢复等。配置域名时无需在 Aws 界面控制台操作，只需用命令操作即可，能在一定程度上提高效率，将工作代码化。
工具地址（Go 语言版）：https://github.com/barnybug/cli53
该工具还有个 Python 版本，是同一个作者，但是 Python 版的已不再维护，目前主要支持 Go 语言版的。
#### 1. 安装
https://github.com/barnybug/cli53

Mac 下安装：
```bash
brew install cli53
```
#### 2. 配置 AWS 访问密钥
- 在控制台新建拥有 Route53 访问权限的 IAM 账号，获取 aws key

![这里写图片描述](/images/cli53.png)

- 将 aws key 添加到环境变量
```bash
export AWS_ACCESS_KEY_ID="xxxx"
export AWS_SECRET_ACCESS_KEY="xxxxx"
```

#### 使用方法总结
1.创建一个域名托管（指定的域名必须是有效的，否则报错）
```bash
cli53 create example.com --comment 'my first zone'
```
2.列出 Rout53 当前所有域名
```bash
cli53 list
```
3.导入 BIND 区域文件（用来做域名迁移）
```bash
cli53 import --file zonefile.txt example.com
```
4.导出域名 BIND 区域文件（用来备份，防止误操作导致不可恢复）
```bash
# 导出非完全符合标准的 bind 文件
cli53 export example.com

# 导出完全符合标准的 bind 文件（一般使用该命令备份）
cli53 export --full example.com
```
5.创建一个 A 记录指向 `192.168.0.1`，并设置 TTL 为 `60s`
```bash
cli53 rrcreate example.com 'www 60 A 192.168.0.1'
```
6.更新上面创建的 A 记录，指向 `192.168.0.2`
```bash
cli53 rrcreate --replace example.com 'www 60 A 192.168.0.2'
```
7.删除一个 A 记录
```bash
cli53 rrdelete example.com www A
```
8.创建一个 MX 记录
```bash
cli53 rrcreate example.com '@ MX 10 mail1.' '@ MX 20 mail2.'
```
9.创建一个轮询的 A 记录
```bash
cli53 rrcreate example.com '@ A 127.0.0.1' '@ A 127.0.0.2'
```
10.创建 CNAME 记录
```bash
cli53 rrcreate example.com 'login CNAME www'
cli53 rrcreate example.com 'mail CNAME ghs.googlehosted.com.'
```
11.创建 ELB 别名记录
```bash
cli53 rrcreate example.com 'www AWS ALIAS A dns-name.elb.amazonaws.com. ABCDEFABCDE false'
```
12.删除一个域名（⚠️危险 删除时如果域名有记录则必须指定 `--purge` 选项）
```bash
cli53 delete --purge example.com
```
13.删除一个域名的所有记录（⚠️危险）
```bash
cli53 rrpurge example.com
```

#### 域名导入注意事项
有的域名提供商，比如 GoDaddy 提供域名记录导出功能，但是导出来后的 BIND 域文件并不是符合标准的，`CNAME`
或者 `MX` 记录末尾没有圆点 `.`，这样在导入 Route53 后会出现问题，需要在导入之前用如下命令处理一下文件
（`MX` 和 `CNAME` 记录末尾添加圆点）再导入：
```bash
perl -pe 's/((CNAME|MX\s+\d+)\s+[-a-zA-Z0-9._]+)(?!.)$/$1./i' broken.txt > fixed.txt
```

