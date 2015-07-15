---
layout: post
title: "redis 主从平滑切换"
date: 2014-10-04 22:28
comments: true
description: Redis slave
categories: DataBase
---

业务原来所在的Redis需要做迁移，感谢Redis，主从切换不算太复杂。
为了可以更好地阐述过程，我们定义了：

* A 服务器，原来拥有 redis 数据库的机器
* B 服务器，未来取代 A 服务器的 redis 数据库服务器


## 主从同步

把 B 服务器的 redis 配置更改为:

```
slaveof  IP地址 端口
```

这样就开始同步了，并且只读了。接下来准备把 B 服务器提升为 master,

```
redis-in-b-host> slaveof no one
```

B服务器就断掉主从同步，提升为主，并行可读写

## Tips;

我们在停止服务的5分钟，在 A 的命令行，执行

```
bgsave
```
使得内存里的数据，完整地保存于 dumps.rdb
