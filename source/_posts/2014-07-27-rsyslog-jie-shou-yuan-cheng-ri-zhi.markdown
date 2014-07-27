---
layout: post
title: "rsyslog 接收远程日志"
date: 2014-07-27 14:26
comments: true
keywords: rsyslog 配置
description: rsyslog 配置
categories: Server
---

Rsyslog 接收远程日志

需要开启运程模式, 以ubuntu为例子

```
vim /etc/default/rsyslog
RSYSLOGD_OPTIONS="-c5 -r -x"
```

编写模板,文档中说到要在`rsyslog.conf`里面编辑

```
vim /etc/rsyslog.conf
$ActionFileDefaultTemplate RSYSLOG_TraditionalFileFormat
$template DynFile, "/data/log/%$year%%$month%%$day%/%$year%%$month%%$day%%$hour%.log"
$template dotalogformat, "%msg%\n"
```

编写过滤规则, 修改 `/etc/rsysconf.d/your_business.conf`

```
# 开通端口
$ModLoad imtcp
$InputTCPServerRun 1514

# 过滤规则
if $msg contains "xx" then ?DynFile;dotalogformat

# 为了不让它写入syslog.log 而直接写入目标模板
:msg, contains, "xx" ~
```

重启服务

```
sudo service rsyslog start
```

