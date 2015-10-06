---
layout: post
title: "Python 使用 LDAP"
date: 2015-10-01 09:49
comments: true
categories: Program
description: python-ldap
---

## 写在开头

过去的两个多星期，几位小伙伴同心协力完成一个自研的，类似[pushover](http://pushover.net)的产品。感谢 Leader 的信任，让我在负责开发的同时也兼顾了一把项目经理。谢谢 IOS, Android 客户端的兄弟，设计师的支持，还有一位实习生，开心看到他的点滴成长。

提下背景，我们之前使用 `pushover` 来做报警推送，但是它对天朝用户不友好，Android 用户需要翻墙才能使用，有时候不稳定，体现为会收不到信息。pushove 需要付费。我们的受众不仅有程序汪，还有运营产品汪，他们需要一款更容易上手的推送软件，至少不需要番羽墙。于是自己开发一个很有必要。

为最大程度降低用户使用门槛，同时保证用户信息的安全，我们用了 LDAP 账户登陆，严格控制权限，以及 HTTPS 协议开发。下面提一提 LDAP 这个东西。

## LDAP 是什么

[LDAP维基百科](https://zh.wikipedia.org/wiki/%E8%BD%BB%E5%9E%8B%E7%9B%AE%E5%BD%95%E8%AE%BF%E9%97%AE%E5%8D%8F%E8%AE%AE)

简单地讲，它是以目录树的方式存放账户信息。

这次项目中，我们不希望用户重新注册账户，而是采用原有的用户体系，这样对单点登录以及权限控制的好处不严而喻, LDAP 协议呼之欲出。


## virtualenv 下安装 python-ldap

我们采用 Python 开发，这就需要 python-ldap 的帮助了, 记下安装笔记的好处是下次不用在此纠结太长时间。

```
apt-get install libsasl2-dev python-dev libldap2-dev libssl-dev
pip install python-ldap
```

## 身份验证

```
# LDAP 服务的域名和端口
SERVER_NAME = 'xxxx'
SERVER_PORT = xxx
try:
    conn = ldap.open(SERVER_NAME, SERVER_PORT)  
    conn.protocol_version = ldap.VERSION3 #设置ldap协议版本 
    username = "user=xxx,dc=company,dc=com" #身份信息
    password = "xxx" #访问密码
    conn.simple_bind_s(username,password) # 开始绑定，验证成功的话不会抛出异常
except ldap.LDAPError, e: #捕获出错信息
    print e
```

## 一点感悟

鄙人觉得，技术领导要带领项目成长, 除了有责任把控项目风险，推进项目进度。
还必须要花很多的心血在驾驭技术上,  身先士卒去调研可行性, 以及做技术攻关，
而非命令式地分配任务，让同事干活, 只问责结果。
否则很容易导致凝聚力不足,团队技术氛围不足，这样的团队易消极，也易滋生失败的项目。
然而在天朝, 很多人存在一个潜意识:写好代码是为了以后不写代码，这种阶级思想让我反感。

以前看过一个新闻, 硅谷在面试技术 VP ，仍然要求其在各位工程师面前手写代码，以此作为面试的重要环节, 不得不点赞。

