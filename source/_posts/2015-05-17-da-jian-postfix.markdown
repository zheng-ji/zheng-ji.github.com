---
layout: post
title: "搭建 Postfix"
date: 2015-05-17 00:25
comments: true
categories: System
description: Postfix
---

我们需要搭建邮件服务，采用Postfix服务, 坑点不少，遂记录。
现在我们决定在 `IP：1.2.3.4` 的机器上部署`Postfix`服务，让它可以发邮件

### 安装 ### 

```
sudo apt-get install postfix
```

### 配置 ### 

编辑 `/etc/postfix/main.cnf`

```
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no
append_dot_mydomain = no
readme_directory = no
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
myhostname = mail.zheng-ji.info 邮箱服务器域名
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = $myhostname
mydestination = mail.zheng-ji.info, localhost.localdomain, localhost
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128, hash:/etc/postfix/access 可以使用这个邮箱服务的外部地址
relay_domains = $mydestination
inet_interfaces = all
inet_protocols = all
```

### 授权 ### 

为了让邮件能真正到达对方邮箱而不被视为垃圾邮件， 我们需要进行DNS权威认证

```
A 记录指向 1.2.3.4
MX 记录也指向 1.2.3.4
TXT 记录 v=spf1 ip4:1.2.3.4 ~all
```

以上操作是防止被认为垃圾邮件。

外部需要访问该Postfix 服务发送邮件，需要有access权限,
编辑/etc/postfix/access, 假设 `IP:5.6.7.8` 的机器想访问该服务

```
5.6.7.8  OK
```

重启并授权生效

```
sudo service postfix restart
postmap access
```
