---
title: Wireshark 抓包理解 HTTPS 协议
date: 2019-05-26 20:58:20
categories: HTTPS
tags: HTTPS
---

### HTTPS 简介
HTTPS（全称：Hypertext Transfer Protocol over Secure Socket Layer）协议是 HTTP 协议的安全版，在 HTTP 应用层和传输层加入了 SSL/TLS 层，确保数据传输的安全性，所以 HTTPS 协议并不是什么新的协议，仅仅是 HTTP 协议和安全协议的组合。

HTTPS 协议主要解决如下三个通信安全问题：
- 窃听风险（eavesdropping）：第三方可以获知通信内容。
- 篡改风险（tampering）：第三方可以修改通信内容。
- 冒充风险（pretending）：第三方可以冒充他人身份参与通信。

HTTPS 通过 SSL/TLS 协议解决了上述三个问题，可以达到：
- 加密数据以防止数据中途被窃取；
- 维护数据的完整性，确保数据在传输过程中不被改变；
- 认证用户和服务器，确保数据发送到正确的客户机和服务器；

既然安全问题是 SSL/TLS 保证的，那么就有必要仔细探索下 SSL/TLS 协议的机制，如下为 HTTPS 通信的整个网络协议栈，其中 SSL/TLS 协议又分为两层：
- 握手协议（SSL Handshake Protocol）：它建立在记录协议之上，用于在实际的数据传输开始前，通讯双方进行身份认证、协商加密算法、交换加密密钥等。
- 记录协议（SSL Record Protocol）：它建立在可靠的传输协议（如TCP）之上，为高层协议提供数据封装、压缩、加密等基本功能的支持。

关于更多 SSL 和 TLS 知识见之前的文章: [基于 OpenSSL 生成自签名证书](https://qhh.me/2019/05/18/%E5%9F%BA%E4%BA%8E-OpenSSL-%E7%94%9F%E6%88%90%E8%87%AA%E7%AD%BE%E5%90%8D%E8%AF%81%E4%B9%A6/)。
![Alt text](/images/ssl-tls.png)

### SSL/TLS 通信过程
开始加密通信之前，客户端和服务器首先必须建立连接和交换参数，这个过程叫做握手（handshake）。SSL/TLS 握手其实就是通过非对称加密，生成对称加密的 session key 的过程。

假定客户端叫做爱丽丝服务器叫做鲍勃，整个握手过程可以用下图说明：
![Alt text](/images/ssl_handshake.png)

整个握手过程通俗地说分为如下五步（真实的过程涉及的细节比这个多）：
- 第一步，爱丽丝给出协议版本号、一个客户端生成的随机数（Client random），以及客户端支持的加密方法。
- 第二步，鲍勃确认双方使用的加密方法，并给出数字证书、以及一个服务器生成的随机数（Server random）。
- 第三步，爱丽丝确认数字证书有效，然后生成一个新的随机数（Premaster secret），并使 用数字证书中的公钥，加密这个随机数，发给鲍勃。
- 第四步，鲍勃使用自己的私钥，获取爱丽丝发来的随机数（即Premaster secret）。
- 第五步，爱丽丝和鲍勃根据约定的加密方法，使用前面的三个随机数，生成"对话密钥"（session key），用来加密接下来的整个对话过程。

![Alt text](/images/ssl_handshake2.png)

SSL 握手的过程为双方发送消息的过程，这里所说的消息并不是一个独立的 TCP 数据包，而是 SSL 协议的术语。根据服务端实现的不同，可能一个 TCP 包中包含多条消息，而不是每条消息单独发送（每条单独发送效率太低），这个我们后面通过 Wireshark 抓包可以看到。

下图为双方握手过程中互相发送的 SSL 消息：
![Alt text](/images/ssl_message.png)


#### 客户端发送的初始消息
##### Client Hello 消息
客户端发送 Client Hello 消息给服务端来初始化会话消息，该消息包含如下信息：
  - Version Number: 客户端发送它所支持的最高 SSL/TLS 版本。版本 2 代表 SSL 2.0，版本 3 代表 SSL 3.0，版本 3.1 代表 TLS。
  - Randomly Generated Data：一个 32 字节的客户端随机数，该随机数被服务端生成通信用的对称密钥（master secret）；
  - Session Identification ：session ID 被客户端用于恢复之前的会话（只有恢复 session 时该字段才有值），这样可以简化 SSL 握手过程，避免每次请求都建立新的连接而握手，握手过程是需要消耗很多计算资源的。已建立的连接信息存储在客户端和服务端各自的 session 缓存中，用 session ID 标识；
  - Cipher Suite: 客户端发送它所支持的加密套件列表；
  - Compression Algorithm: 压缩算法，目前该字段几乎不在使用；

#### 服务端响应
##### Server Hello 消息
服务端回复 Server Hello 消息给客户端：
- Version Number：服务端发送双发所支持的最高的 SSL/TLS 版本；
- Randomly Generated Data：一个 32 字节的服务端随机数，被客户端用于生成通信用的对称密钥（master secret）；
- Session Identification：该字段有如下三中情况：
 - New session ID：客户端和服务端初次建立连接时生成的 session ID。或者客户端尝试恢复 session，但是服务端无法恢复，因此也会生成新的 session ID；
 - Resumed Session ID：和客户端发送的恢复会话 ID 一致，用于恢复会话；
 - Null：该字段为 Null，表明这是一个新的 Session，但是服务端不打算用于后续的会话恢复，因此不会产生 session ID，该字段为空；
- Cipher Suite: 服务端发送双发支持的最安全的加密套件；
- Compression Algorithm：指定双方使用的压缩算法，目前该字段几乎不在使用；

##### Server Certificate 消息
服务端发送自己的 SSL 证书给客户端，证书中包含服务端的公钥，客户端用该证书验证服务端的身份。

##### Server Key Exchange 消息
这个消息是可选的，该消息主要用来传递双方协商密钥的参数，比如双方使用 Diffie-Hellman (迪菲) 算法生成 premaster secret 时，会用该字段传递双方的公共参数。所以具体该字段是什么内容取决于双方协商密钥的加密套件。

##### Client Certificate Request 消息
这个消息也是可选的，只有当服务端也需要验证客户端会用到。有的安全度高的网站会要求验证客户端，确认客户的真实身份，比如银行之类的网站。

##### Server Hello Done 消息 
服务器发送 ServerHelloDone 消息，告知客户端服务端这边握手相关的消息发送完毕，等待客户端响应。

#### 客户端回复
##### Client Certificate 消息
如果服务端发送了 Client Certificate Request 消息，那么客户端会发送该消息给服务端，包含自己的证书信息，供服务端进行客户端身份认证。
##### Client Key Exchange 消息
根据协商的密钥算法不同，该消息的内容会不同，该消息主要包含密钥协商的参数。比如双方使用 Diffie-Hellman (迪菲) 算法生成 premaster secret 时，会用该字段传递双方的公共参数。
##### Certificate Verify 消息
该消息只有在 Client Certificate message 消息发送时才发送。客户端通过自己的私钥签名从开始到现在的所有发送过的消息，然后服务端会用客户端的公钥验证这个签名。

##### Change Cipher Spec 消息
通知服务器此消息以后客户端会以之前协商的密钥加密发送数据。

##### Client Finished 消息
客户端计算生成对称密钥，然后使用该对称密钥加密之前所有收发握手消息的 Hash 值，发送给服务器，服务器将用相同的会话密钥（使用相同方法生成）解密此消息，校验其中的Hash 值。该消息是 SSL 握手协议记录层加密的第一条消息。

#### 服务端最后对客户端响应
##### Change Cipher Spec  消息
通知客户端此消息以后服务端将会以之前协商的密钥加密发送数据。

##### Server Finished 消息
服务器使用对称密钥加密（生成方式与客户端相同）之前所发送的所有握手消息的hash值，发送给客户端去校验。

至此 SSL 握手过程结束，双发之后的通信数据都会用双方协商的对称密钥 Session Key 加密传输。

下图为 SSL/TLS 通信的整个过程：TCP 三次握手 + SSL/TLS 握手：
![Alt text](/images/ssl_handshake3.png)

### Wireshark 抓包分析 SSL/TLS 握手过程
本节使用 wireshark 抓包工具分析一个完整的 HTTPS 通信过程，看看通信过程中双方消息是如何传送的。前面我们说过，根据服务端实现的不同，可能一个 TCP 包中包含多条 SSL/TLS 消息，而不是每条消息单独发送（每条单独发送效率太低）。

使用如下 wireshark https 包过滤器:
```
tcp.port==443 and (ip.dst==104.18.40.252 or ip.src==104.18.40.252)
```
下面为 Wireshark 抓取的 https 流量包，展示了整个通信过程：建立 TCP 连接 --> SSL/TLS 握手 --> 应用数据加密传输：
![Alt text](/images/wireshark_ssl_handshake1.png)

上面是一个实际的 SSL/TLS 握手过程，分为如下 5 步：
1. 客户端发送 Client Hello 消息给服务端；
2. 服务端回应 Server Hello 消息；
3. 服务端同时回应 Server Certificate、Server Key Exchange 和 Server Hello Done 消息；
4. 客户端发送 Client Key Exchange、Change Cipher Spec 和 Client Finished 消息；
5. 服务端最后发送 Change Cipher Spec 和 Server Finished 消息；

下面我们分步分析每个阶段的包的内容，看是否和前面的理论一致。
#### 客户端发送 Client Hello 消息给服务端
![Alt text](/images/wireshark_ssl_handshake2.png)

可以看出 TLS 协议确实分为两层：TLS 记录层、TLS 握手层，其中 TLS 握手层基于 TLS 记录层。

另外客户端发送的 Client Hello 消息当中包含的信息也可以看到：
- Version：客户端支持的 TLS 版本号；
- Random：客户端生成的 32 字节随机数；
- Session ID：会话 ID；
- Cipher Suites：客户端支持的加密套件列表；
- Compression Methods：客户端支持的压缩算法；

#### 服务端回应 Server Hello 消息
![Alt text](/images/wireshark_ssl_handshake3.png)

Server Hello 包含如下信息：
- Version：双方支持的 TLS 版本号；
- Random：服务端生成的 32 字节随机数；
- Session ID：会话 ID；
- Cipher Suites：双方协商的加密套件；
- Compression Methods：压缩算法；

#### 服务端同时回应 Server Certificate、Server Key Exchange 和 Server Hello Done 消息
![Alt text](/images/wireshark_ssl_handshake4.png)

可以看出每个 TLS 记录层是一个消息，服务端同时回复了有 3 个消息：Server Certificate、Server Key Exchange、Server Hello Done。

从 Server Key Exchange 消息可以看出双方密钥协商使用的是 Diffie-Hellman (迪菲) 算法，该消息用于传递 Diffie-Hellman (迪菲) 算法的参数。

#### 客户端发送 Client Key Exchange、Change Cipher Spec 和 Client Finished 消息
![Alt text](/images/wireshark_ssl_handshake5.png)

可以看出客户端同时回复了 3 个消息：Client Key Exchange、Change Cipher Spec 和 Client Finished 消息。Client Key Exchange 的内容为 Diffie-Hellman (迪菲) 算法的参数，用于生成 premaster key，然后和双方之前的随机数结合生成对称密钥。

#### 服务端最后发送 Change Cipher Spec 和 Server Finished 消息
![Alt text](/images/wireshark_ssl_handshake6.png)

 服务端最后发送 Change Cipher Spec 和 Server Finished 消息，至此 SSL/TLS 握手完毕，接下来双方会用对称加密的方式加密传输数据。

### 相关资料
https://segmentfault.com/a/1190000002554673 | SSL/TLS 原理详解
http://www.ruanyifeng.com/blog/2014/02/ssl_tls.html | SSL/TLS 协议运行机制概述
http://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html | 图解 SSL/TLS 协议
https://www.jianshu.com/p/a3a25c6627ee | Https详解+wireshark抓包演示
https://segmentfault.com/a/1190000007283514 | TLS/SSL 高级进阶
https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc785811(v=ws.10) | 微软 Windows 文档
