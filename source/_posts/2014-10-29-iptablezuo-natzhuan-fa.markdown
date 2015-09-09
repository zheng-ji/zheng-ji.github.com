---
layout: post
title: "iptable 之 NAT 转发"
date: 2014-10-29 22:38
comments: true
keywords: NAT iptables
description: NAT iptables
categories: System
---

简单简述下我遇到的问题:

现在局域网有2台机器, 其中一台机器(下文我们称之为ServerA)可以访问外网,有独立IP,而另外一台机器(ServerB)访问不了外网, 需要想办法让 ServerB 也能上网。

* ServerA 操作

```
vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
iptables -t nat -A POSTROUTING -s 10.4.0.0/16 -j MASQUERADE
```


ServerB

编辑 /etc/network/interface
主要是修改gateway 参数，指向 ServerA 的 IP

```
sudo ifdown eth0; sudo ifup eth0 ``` 
ServerB 就可以连接外网了。
