---
title: 基于 OpenSSL 生成自签名证书
date: 2019-05-18 12:41:23
categories: HTTPS
tags: HTTPS
---

### <p style="color:green;">PKI、CA、SSL、TLS、OpenSSL几个概念</p>

#### <p style="color:green;"> PKI 和 CA</p>
PKI 就是 Public Key Infrastructure 的缩写，翻译过来就是公开密钥基础设施。它是利用公开密钥技术所构建的，解决网络安全问题的，普遍适用的一种基础设施。

PKI 是目前唯一的能够基本全面解决安全问题的可能的方案。 PKI 通过电子证书以及管理这些电子证书的一整套设施，维持网络世界的秩序；通过提供一系列的安全服务，为网络电子商务、电子政务提供有力的安全保障。

通俗点说 PKI 就是一整套安全相关标准，然后基于这套标准体系衍生一系列安全相关的产品，主要目的是保证数据在网络上安全、可靠地传输。

PKI 主要由以下组件组成：
- 认证中心 CA(证书签发) ;
- X.500目录服务器(证书保存) ;
- 具有高强度密码算法(SSL)的安全WWW服务器(即配置了 HTTPS 的 apache) ;
- Web(安全通信平台): Web 有 Web Client 端和 Web Server 端两部分
- 自开发安全应用系统 自开发安全应用系统是指各行业自开发的各种具体应用系统，例如银行、证券的应用系统等。

CA 是 PKI 的"核心"，即数字证书的申请及签发机关，CA 必须具备权威性的特征，它负责管理 PKI 结构下的所有用户(包括各种应用程序)的证书，把用户的公钥和用户的其他信息捆绑在一起，在网上验证用户的身份，CA 还要负责用户证书的黑名单登记和黑名单发布 。

CA 实现了 PKI 中一些很重要的功能：
- 接收验证最终用户数字证书的申请；
- 确定是否接受最终用户数字证书的申请-证书的审批；
- 向申请者颁发、拒绝颁发数字证书-证书的发放；
- 接收、处理最终用户的数字证书更新请求-证书的更新；
- 接收最终用户数字证书的查询、撤销；
- 产生和发布证书废止列表(CRL)；
- 数字证书的归档；
- 密钥归档；
- 历史数据归档；

在这么多功能中，CA 的核心功能就是"发放"和"管理"数字证书：
![Alt text](/images/ca-arch.png)

#### <p style="color:green;">SSL 和 TLS</p>
SSL 和 TLS 协议是介于 HTTP 协议与 TCP 之间的一个可选层，主要用于 Web 客户端和服务器之间进行数据的安全传输：
![Alt text](/images/ssl-tls.png)
- **SSL:** Secure Socket Layer，安全套接字层），为Netscape所研发，用以保障在Internet上数据传输之安全，利用数据加密(Encryption)技术，可确保数据在网络上之传输过程中不会被截取。当前版本为3.0。它已被广泛地用于Web浏览器与服务器之间的身份认证和加密数据传输。SSL协议位于TCP/IP协议与各种应用层协议之间，为数据通讯提供安全支持。SSL协议可分为两层： 
SSL记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。 
SSL握手协议（SSL Handshake Protocol）：它建立在SSL记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。
- **TLS:** (Transport Layer Security，传输层安全协议)，用于两个应用程序之间提供保密性和数据完整性。
TLS 1.0是IETF（Internet Engineering Task Force，Internet工程任务组）制定的一种新的协议，它建立在SSL 3.0协议规范之上，是SSL 3.0的后续版本，可以理解为SSL 3.1，它是写入了 RFC 的。该协议由两层组成： TLS 记录协议（TLS Record）和 TLS 握手协议（TLS Handshake）。较低的层为 TLS 记录协议，位于某个可靠的传输协议（例如 TCP）上面。

SSL/TLS协议提供的服务主要有：
- 认证用户和服务器，确保数据发送到正确的客户机和服务器；
- 加密数据以防止数据中途被窃取；
- 维护数据的完整性，确保数据在传输过程中不被改变；

#### <p style="color:green;">OpenSSL</p>
![Alt text](/images/openssl.png)
[OpenSSL](https://github.com/openssl/openssl) 是一个开源的加密工具包，主要包括如下三部分：
- **libssl (with platform specific naming)**:
     Provides the client and server-side implementations for SSLv3 and TLS.
- **libcrypto (with platform specific naming)**:
     Provides general cryptographic and X.509 support needed by SSL/TLS but
     not logically part of it.
- **openssl**:
     A command line tool that can be used for:
        Creation of key parameters
        Creation of X.509 certificates, CSRs and CRLs
        Calculation of message digests
        Encryption and decryption
        SSL/TLS client and server tests
        Handling of S/MIME signed or encrypted mail
        And more...
        
### <p style="color:green;">使用 OpenSSL 生产自签名 SSL 证书过程</p>
![Alt text](/images/gen-certificate.png)

以下为 Centos7 环境下生成自签名 SSL 证书的具体过程：
1. 修改 openssl 配置文件
```
vi /etc/pki/tls/openssl.cnf
# match 表示后续生成的子证书的对应项必须和创建根证书时填的值一样，否则报错。以下配置只规定子证书的 countryName 必须和根证书一致。
[ policy_match ] 段配置改成如下：
countryName             = match
stateOrProvinceName     = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
```

2. 在服务器 pki 的 CA 目录下新建两个文件
```
cd /etc/pki/CA && touch index.txt serial && echo 01 > serial
```
3. 生成 CA 根证书密钥
```
cd /etc/pki/CA/ && openssl genrsa -out private/cakey.pem 2048 && chmod 400 private/cakey.pem
```
4. 生成根证书（根据提示输入信息，除了 Country Name 选项需要记住的，后面的随便填）
```
openssl req -new -x509 -key private/cakey.pem -out cacert.pem
```
5. 生成密钥文件
```
openssl genrsa -out nginx.key 2048
```
6. 生成证书请求文件（CSR）: 
A. 根据提示输入信息，除了 Country Name 与前面根证书一致外，其他随便填写
B. Common Name 填写要保护的域名，比如：*.qhh.me
```
openssl req -new -key nginx.key -out nginx.csr
```
7. 使用 openssl 签署 CSR 请求，生成证书
```
openssl ca -in nginx.csr -cert /etc/pki/CA/cacert.pem -keyfile /etc/pki/CA/private/cakey.pem -days 365 -out nginx.crt

参数项说明：
-in: CSR 请求文件
-cert: 用于签发的根 CA 证书
-keyfile: 根 CA 的私钥文件
-days: 生成的证书的有效天数
-out: 生成证书的文件名
```
至此自签名证书生成完成，最终需要：nginx.key 和 nginx.crt

### <p style="color:green;">配置 Nginx 使用自签名证书</p>
```
server {
        listen  80;
        server_name     domain;
        return  301     https://$host$request_uri;
}
server {
        listen  443 ssl;
        ssl_certificate       ssl/nginx.crt; # 前面生成的 crt 证书文件
        ssl_certificate_key   ssl/nginx.key; # 前面生成的证书私钥
        server_name     domain;
        location / {
            root /var/www-html;
            index  index.html;
        }
}
```

### <p style="color:green;">相关资料</p>
http://seanlook.com/2015/01/18/openssl-self-sign-ca/ | 基于 OpenSSL 自签署证书
http://www.cnblogs.com/littlehann/p/3738141.html | openSSL命令、PKI、CA、SSL证书原理
https://cnzhx.net/blog/ssl-on-lamp-on-vps/
http://seanlook.com/2015/01/15/openssl-certificate-encryption/ | OpenSSL 与 SSL 数字证书概念贴
https://kb.cnblogs.com/page/194742/ | 数字证书及 CA 的扫盲
http://netsecurity.51cto.com/art/200602/21066.htm | PKI/CA 技术的介绍
http://cnzhx.net/blog/self-signed-certificate-as-trusted-root-ca-in-windows/ | 浏览器添加自签名证书
https://aotu.io/notes/2016/08/16/nginx-https/index.html | Nginx 配置 HTTPS 服务器
