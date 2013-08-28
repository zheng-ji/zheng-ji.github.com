---
layout: post
title: "Redis是个好东西"
date: 2013-02-17 14:10
comments: true
categories: server
---
 :) 最近主动请缨做到一个关于Redis的东西，甚有收获，与大家分享：- ）

 以下选自百度百科的简介

redis是一个key-value存储系统。和Memcached类似，它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)和zset(有序集合)。这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，redis支持各种不同方式的排序。与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。Redis 是一个高性能的key-value数据库。 redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Python，Ruby，Erlang，PHP客户端，使用很方便。

Redis提供了很多高级的数据结构：哈希表，列表，集合，字典，这些东西看上去很好吃的样子

#### 想必你也在思考了，那这东西可以用在什么场景呢，

Redis作者自己是这样说的：
+ 取最新N个数据的操作

比如典型的取你网站的最新文章，通过下面方式，我们可以将最新的5000条评论的ID放在Redis的List集合中，并将超出集合部分从数据库获取

+ 排行榜应用，取TOP N操作

这个需求与上面需求的不同之处在于，前面操作以时间为权重，这个是以某个条件为权重，比如按顶的次数排序，这时候就需要我们的sorted set出马了，将你要排序的值设置成sorted set的score，将具体的数据设置成相应的value，每次只需要执行一条ZADD命令即可。

+ 需要精准设定过期时间的应用

比如你可以把上面说到的sorted set的score值设置成过期时间的时间戳，那么就可以简单地通过过期时间排序，定时清除过期数据了，不仅是清除Redis中的过期数据，你完全可以把Redis里这个过期时间当成是对数据库中数据的索引，用Redis来找出哪些数据需要过期删除，然后再精准地从数据库中删除相应的记录。

+ 计数器应用

Redis的命令都是原子性的，你可以轻松地利用INCR，DECR命令来构建计数器系统。

+ Uniq操作，获取某段时间所有数据排重值

这个使用Redis的set数据结构最合适了，只需要不断地将数据往set中扔就行了，set意为集合，所以会自动排重。

+ 实时系统，反垃圾系统

通过上面说到的set功能，你可以知道一个终端用户是否进行了某个操作，可以找到其操作的集合并进行分析统计对比等。没有做不到，只有想不到。

+ Pub/Sub构建实时消息系统

Redis的Pub/Sub系统可以构建实时的消息系统，比如很多用Pub/Sub构建的实时聊天系统的例子。

+ 构建队列系统

使用list可以构建队列系统，使用sorted set甚至可以构建有优先级的队列系统。

+ 缓存

这个不必说了，性能优于Memcached，数据结构更多样化。

我用到的场景是
>python + redis + msgpack

redis-py是个好东西
```
import redis
#db是选择的数据库，Redis默认有16个库
r = redis.Redis(host='localhost', port=6379, db=0)
keylist = r.keys("*")
for key in keylist:
    record = r.get(key)
```

后话:
msgpack简介
MessagePack是一个基于二进制高效的对象序列化Library用于跨语言通信。
它可以像JSON那样，在许多种语言之间交换结构对象；但是它比JSON更快速也更轻巧。
支持Python、Ruby、Java、C/C++、Javascript等众多语言。
Happy Hacking
