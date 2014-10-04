---
layout: post
title: "redis热备"
date: 2014-10-04 22:28
comments: true
description: Redis Replication
categories: DataBase
---

一转眼到了 10 月份了, 依然很忙, 好多碰到的，解决了的, 没能解决的蛋疼问题, 那些一闪而过的被记载于 evernote里, 信誓旦旦说过要记载下来，终究也还是被时间欺骗。 

在过去的九月份，我们迁移了一批业务到云上。除了 mysql 需要热备处理，redis-server 同样也面临这种问题。相比起来，redis 的热备显得更为简单。

为了可以更好地阐述过程，我们定义了三个角色

+ A 服务器，原来拥有 redis 数据库的机器 
+ B 服务器，未来取代 A 服务器的 redis 数据库服务器
+ C 服务器，B 服务器的 slave

* 我们在停止服务的5分钟，在 A 的命令行，执行

```
bgsave
```

使得内存里的数据，完整地保存于 dumps.rdb

* 配置好 B 服务器的redis server, 用上述的 dumps.rdb 取代该服务器的 dumps.rdb , 这里说的配置好，需要指定好 bind-address, 推荐使用内网 IP, 备份目录

* 在 C 服务器的 redis 配置上 更改为:

```
slaveof  IP地址 端口
slave-serve-stale-data no
slave-read-only yes # 只读
```

这样就开始备份了。
