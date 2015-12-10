---
layout: post
title: "Nginx 日志利器 GoAccess"
date: 2015-12-10 23:28
comments: true
categories: System
description: GoAccess Nginx
---

我们经常需要从 Nginx 日志分析得出很多有价值的东西，分析的方法是各种 shell, awk, python, 现在 [GoAccess](https://github.com/allinurl/goaccess) 给你另外一种选择, 值得拥有。

* 安装
用以下的方式安装，才能得到新版的 GoAccess, 功能更健全
 
```
$ echo "deb http://deb.goaccess.io $(lsb_release -cs) main"|sudo tee -a /etc/apt/sources.list
$ wget -O - http://deb.goaccess.io/gnugpg.key | sudo apt-key add -
$ sudo apt-get update
$ sudo apt-get install goaccess
```
 
* 推荐配置
 
安装完成之后，会生成一份 `/etc/goaccess.conf` 稍作编辑，这就是默认的配置，免去了后续每次都要定义格式

```
time-format %T   # 只有这种方式才能解决 0.0us 的显示问题
date-format %d/%b/%Y
log-format %h %^[%d:%t %^] "%r" %s %b "%R" "%u" %T
```
 
* 使用

输出报表，报表中，我们可以看到最常访问的 IP, 接口，以及每个接口使用带宽，平均响应时长，状态码等，对业务分析有较好的便利性
 
```
终端显示
goaccess -f access.log 
 
输出报表，报表中，我们可以看到top ip, 接口，以及接口使用带宽，平均响应时长，状态码等
goaccess -f access.log > report.html
 
具体某段时间的输出
sed -n '/5\/Dec\/2015/,/10\/Dec\/2010/ p' access.log | goaccess -a
 
处理已经压缩的日志
zcat access.log.*.gz | goaccess
```

