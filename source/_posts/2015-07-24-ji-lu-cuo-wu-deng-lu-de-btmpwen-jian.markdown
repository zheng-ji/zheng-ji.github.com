---
layout: post
title: "记录错误登陆的 btmp 文件"
date: 2015-07-24 23:06
comments: true
categories: System
description: linux
---

今天查看了服务器时，发现 `/var/log/btmp` 日志文件较大。

此文件是记录错误登录的日志， 文件较大意味着有人使用密码字典登录ssh服务，
这个文件是需要用 `lastb` 命令才可读的。

查看尝试恶意登陆的前十个IP

```
sudo lastb | awk '{ print $3}' | awk '{++S[$NF]} END {for(a in S) print a, S[a]}' | sort -rk2 |head
```

如果有有必要封阻IP的话，可以执行：

```
iptables -A INPUT -i eth0 -s *.*.*.0/24 -j DROP
```
