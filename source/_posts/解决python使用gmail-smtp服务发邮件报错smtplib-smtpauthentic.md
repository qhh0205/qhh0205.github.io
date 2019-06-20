---
title: 解决python使用gmail smtp服务发邮件报错smtplib.smtpauthentic
date: 2018-02-17 11:21:43
categories: Python
tags: Python
---

#### 前言
之前使用python发gmail邮件时也遇到过同样的问题，当时在网上找了很多教程鼓捣了半天终于可以发出邮件了，也没搞明白什么原因。如今换了个gmail账号后同样的问题又复现了，又查了半天终于搞定了，不过这次问题还比较奇怪，根据网上很多教程做了后发现邮件是可以发送了，但是在阿里云机器上可以发送，到了AWS机器就不行了，还是报同样的错误，这次终于搞明白什么原因了，在此mark一下，方便后面遇到同样的问题可以快速解决。
#### 问题描述
执行python邮件发送代码后报错如下：
```bash
smtplib.SMTPAuthenticationError: (534, '5.7.14 <https://accounts.google.com/signin/continue?sarp=1&scc=1&plt=AKgnsbuK\n5.7.14 
Gn6F1ag1icz1n3PPXqxUPRGkDb09Kdq88-uxXe-1Tcmguc_HnivrUAD3tjx_1Va5SWXbWQ\n5.7.14 FX-6LHu7-lu5-
gPGT3yDrlqNG6gZUTiqk7zvxe1Mv6LgbAtipnqFDdLcvcBbvP8vy0scEd\n5.7.14 llhSJfaRjhTyXG--HuJ6YhsdCkyDA3O8SjWjv9JFORNiXab2Mp5OnJ1NLtIie1saDnQs_X\n5.7.14 
PLuHe5XRB2BofVVItB8Gg9VrxKqpc> Please log in via your web browser and\n5.7.14 then try again.\n5.7.14  Learn more at\n5.7.14 
https://support.google.com/mail/answer/78754 u131sm18903705pgc.89 - gsmtp')
```
python gmail邮件发送代码如下:
```bash
#/data/xce/local//python/bin/python2.7
# -*- coding:utf-8 -*-

import smtplib
import base64
from email.mime.text import MIMEText
from email.header import Header
from email.mime.multipart import MIMEMultipart
from email.mime.image import MIMEImage
from email.utils import COMMASPACE

SENDER = 'xxxxx@gmail.com'
SMTP_SERVER = 'smtp.gmail.com'
USER_ACCOUNT = {'username':'xxxxx@gmail.com', 'password':'xxxxxx'}
SUBJECT = "Test Test"


def send_mail(receivers, text, sender=SENDER, user_account=USER_ACCOUNT, subject=SUBJECT):
    msg_root = MIMEMultipart()  # 创建一个带附件的实例
    msg_root['Subject'] = subject  # 邮件主题
    msg_root['To'] = COMMASPACE.join(receivers)  # 接收者
    msg_text = MIMEText(text, 'html', 'utf-8')  # 邮件正文
    msg_root.attach(msg_text)  # attach邮件正文内容

    smtp = smtplib.SMTP('smtp.gmail.com:587')
    smtp.ehlo()
    smtp.starttls()
    smtp.login(user_account['username'], user_account['password'])
    smtp.sendmail(sender, receivers, msg_root.as_string())

if __name__=="__main__":
    send_mail(['codenutter@foxmail.com'], "Test Test")
```
#### 解决问题
**出现这个错误的原因有两个：**
  - Google阻止用户使用不符合他们安全标准的应用或设备登陆gmail
  [https://support.google.com/accounts/answer/6010255?hl=zh-Hans](https://support.google.com/accounts/answer/6010255?hl=zh-Hans)
  - Gmail没有解除验证码认证
 
**知道原因了就可以有效地解决问题了，解决方法如下：**
  - 允许不够安全的应用使用您的账号:点击如下链接，开启“允许不够安全的应用”功能 [https://myaccount.google.com/lesssecureapps](https://myaccount.google.com/lesssecureapps)
  ![](/images/py-gmail1.png)
  - 解除gmail验证码认证:
  点击如下链接，然后点击继续即可
  [https://accounts.google.com/b/0/displayunlockcaptcha ](https://accounts.google.com/b/0/displayunlockcaptcha )
  ![](/images/py-gmail2.png)

#### 参考文章
[https://segmentfault.com/q/1010000008458788/a-1020000008470509](https://segmentfault.com/q/1010000008458788/a-1020000008470509)
[https://blog.user.today/gmail-smtp-authentication-required](https://blog.user.today/gmail-smtp-authentication-required)
[https://stackoverflow.com/questions/35659172/django-send-mail-from-ec2-via-gmail-gives-smtpauthenticationerror-but-works
](https://stackoverflow.com/questions/35659172/django-send-mail-from-ec2-via-gmail-gives-smtpauthenticationerror-but-works)
