---
layout: post
title: "MiniGame 服务端设计"
date: 2013-08-20 18:54
comments: true
categories: Server
---

  最近和小伙伴们在做一个游戏。有机会单挑服务端了。被信任是有压力也有动力 ：），可以尝试各种好玩的技术的感觉是很爽。Jerry 和Cat 所主导的客户端，Mzt的美术都很给力。作为云风的忠实粉，可以独立设计游戏服务器后台，我对这样的挑战期待已久。

#### 逻辑架构图
{% img /images/2013/08/minigame.png %}

主要由三大逻辑层组成：
+ 连接（接入）服务器模块:   ConnServer
+ 游戏服务器模块 ：        GameServer
+ 缓存服务器模块：         CacheServer
  (附：我们自己编写了静态资源的服务器功能，提供给用户下载Apk的功能)

* 接入服务器模块
服务器Socket建立，建立玩家通讯连接，为每一位连上服务器的玩家建立一个线程。
服务器维护一个连接列表。采用map 的方式记录每一位玩家的连接信息。

* 游戏服务器模块：
    这是处理游戏业务逻辑的主要模块。它包括了消息包解析器服务（Parser），游戏大厅服务（GameRoom），账户体系服务（Account）,消息中心服务（MsgCenter）,心跳服务（Heartbeat）.日志功能模块（log）

+ 消息包解析器服务（Parser）:主要实现了消息状态机。解析收到的客户端的数据包的协议号，然后将消息路由到不同功能模块去处理。
+ 账户体系服务（Account）：提供用户注册和登陆管理的模块
+ 游戏大厅服务（GameRoom）：采用Redis 内存数据库，构造消息队列。模拟游戏大厅，并对玩家进行配对筛选，构造set类型的匹配对完成1对1 的对战模式
+ 消息中心服务（MsgCenter）:完成游戏过程中玩家信息的交互。
+ 心跳服务（Heartbeat）：这是一个相对独立的进程，每7秒遍历用户信息，完成心跳检测，清理无效的过期数据，保持游戏数据的一致性.
+ 日志功能：打印程序中的日志。
+ 静态资源下载器（downloader）：采用tornado拓展编写web服务，提供Apk静态资源下载的模块

* 缓存服务器模块
    现阶段游戏对服务的要求不需要做到持久化的保存，于是服务器没有采用mysql之类的范式数据库。我们的数据收发一直处于频繁状态，于是我们采用了Redis这种分布式的内存数据库来实现我们的数据缓存模块，后续会介绍数据的键值设计

#### 类图
{% img /images/2013/08/class.png %}

* cache数据库的设计：
利用Redis的数据特性,新增一个用户保证用户ID的原子性。
+ counter: 对新用户的注册执行自增操作，从而保证账号Id 唯一

```
说明 key value
实例 counter 1
```

* 用户账号：
Key设计为 user：name:用户名’，Value用户ID，
为保证用户可以同时通过id 查找名字 ，也设计了 ‘user:id:ID值’值为用户名

```
说明 key                    value
实例 User:name:zhengji         1
实例 User:id:1                 zhengji
```

* 玩家通讯缓存设计：
Key值：user:用户名：msg: Value:‘分数，buff_id,时间戳’

```
说明 key              value
实例 user:zhengji:msg 100, 2,   1324256778.89
```

* 玩家匹配对设计
在游戏大厅的功能里，我们采用了redis的list模拟了玩家等待队列的功能 在真正匹配到玩家对的时候，使用table_xx 来记录玩家匹配信息,它是一个set的类型,在游戏退出的时候会被驱动去清除该表的信息

```
说明   key      value
实例 Table_1 ‘jerry,zhengji’
```
       
为标志每个玩家的table号，我们使用了另外一个键值对
Table:玩家名：tableindex.

```
说明   key       value
实例 Table:zhengji 1
```

