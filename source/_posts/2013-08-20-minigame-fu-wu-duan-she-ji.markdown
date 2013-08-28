---
layout: post
title: "MiniGame 服务端设计"
date: 2013-08-20 18:54
comments: true
categories: Server
---

  最近和小伙伴们在做一个游戏。有机会单挑服务端了。被信任是有压力也有动力 ：），可以尝试各种好玩的技术的感觉是很爽。Jerry 和Cat 所主导的客户端，Mzt的美术都很给力。作为云风的忠实粉，可以独立设计游戏服务器后台，我对这样的挑战期待已久。
##数据库设计

使用Redis架设服务器数据库。就涉及到数据库的设计。利用Redis的数据特性保证新增一个用户保证用户ID的原子性。

```python
INCR counter
```

Redis默认的设计风格习惯采用 表名：主键：列值。做成一个帐户体系

```python
User:name:zheng-ji id
```

以下是帐号体系的代码

``` python
def addAccount(r,name):
        if r.exists('counter'):
            r.incr('counter')
        else:
            r.set('counter',1)
            id = r.get('counter')
        key = "user:name:%s"%name
        r.set(key,id)

    key = "user:id:%s"%id
        r.set(key,name)
    return id
```
    协议设计

    考虑到网络粘包,解包问题，基于应用程序自己设计了包协议
    Len + Header + Msg ，
    采用简单的json打包发送。其实想用msgpack压缩的，但是后来因为客户端的开发时间太短而作废。

    [code]
    Len ：    # 包长度
    Header：
     [
          Action: 4 #Wait
               From:   id #用户ID
                 ]
                 Msg:
                  [
                      "Type":     # 0 表示buf 或者 1：表示score
                         “Score”
                          ]
                          [/code]

                          网络通讯

                          通讯采用TCP协议。最开始想采用Tornado的TCPServer 稍加改装，后来发现可拓展性差，于是就自己写了个TCPServer，支持广播消息（BroadCast）,1 Vs 1通信，回响等服务.
                          没有用到Select这些IO复用，而是给每个连接创建子线程。永真循环用Rec阻塞连接。比较挫 ：（
                          排行榜

                          这个功能很简单的，纯粹地被动推消息。Redis 的存放消息格式

                          [code]
# 10表示用户ID 100 分数
                          Score：user:10 100
                          [/code]

                          玩家联机

                          设计中采用Id对应一个Conn(连接sock句柄)，为了方便查询，同时设计一个Conn对应一个ID，相当于游戏大厅功能。维护一个等待对战的Set，类似游戏大厅，采用Random模块随机匹配，随机匹配两个通讯句柄对。之后进行1对1通信。游戏中的1v1模式，1个玩家需要实时看到对方的游戏状态，服务器做的事情就是将竞争双方的消息互发。这个过程采取的策略是：如果游戏玩家点击联机模式，服务器自动匹配。从等待对战的Set（）里面随机挑取。然后下发对手ID给客户端。以后客户端每次发包都会携带对方的ID，以便在服务器中搜索对方的通讯句柄，服务器同时也将上传的消息发送给对手。从而实现同步，这过程中是可以容忍小许延迟的。
                          可以开拓的点

                              心跳检测来测试客户端是否掉线
                                  掉线如何保持连接
                                      如果是实时性很高的MMORPG的游戏，同步是很重要的，同步 预测算法如何实现

                                      Happy Hacking ! Enjoy The Show :)
