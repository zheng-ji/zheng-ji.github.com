---
layout: post
title: "Twemproxy 一个 Redis 代理"
date: 2015-08-16 12:11
comments: true
categories: Server
description: redis proxy twemproxy 
---

为解决线上 Redis 服务直连出现链接数爆棚而做的调研， 对 Twitter 开源的 twemproxy 做一些记录。 我们之所以放弃官方的 RedisCLuster 是因为不太满意其性能

: [初窥原理](#第一节)
* [安装与配置](#第二节)
* [不支持的操作](#第三节)
* [压力测试](#第四节)
* [摘自极光博客的评论](#第五节)

<h3 id="第一节">初窥原理</h3>

* Twitter 出品的轻量级 Redis，memcached 代理，使用它可以减少缓存服务器的连接数，并且利用它来作分片。
* 作是说最差情况下，性能损耗不会多于20%。背后是用了pipeline，redis是支持使用pipeline批处理的。
* twemproxy 与每个 redis 服务器都会建立一个连接，每个连接实现了两个 FIFO 的队列， 通过这两个队列实现对 redis 的 pipeline 访问，将多个客户端的访问合并到一个连接，这样既减少了redis服务器的连接数，又提高了访问性能。

<h3 id="第二节">安装与配置</h3>

* 安装

```
apt-get install automake
apt-get install libtool
git clone git://github.com/twitter/twemproxy.git
cd twemproxy
autoreconf -fvi
./configure
make
sudo make install
```
默认的可执行文件在 /usr/local/sbin/nutcracker

* 配置文件 /etc/nutcracker/nutcracker.yml

```
alpha:
    listen: 127.0.0.1:8877
    hash: fnv1a_64
    distribution: ketama
    auto_eject_hosts: true
    redis: true
    server_retry_timeout: 30000
    server_failure_limit: 3
    servers:
        - 127.0.0.1:6379:1 master0  #后端的redis-server
        - 127.0.0.1:6380:1 master1
```

当 redis 做缓存的使用的时候应该启用 auto_eject_hosts， 如果某个节点失败的时候将该节点删除，虽然丧失了数据的一致性，但作为缓存使用，保证了这个集群的高可用性。当redis做存储的使用时为了保持数据的一致性，应该禁用 auto_eject_hosts,也就是当某个节点失败之后并不删除该节点。

<h3 id="第三节">不支持的操作</h3>

```
keys command: keys,migrate,move object,randomkey,rename,renamenx,
sort strings command: bitop,mset,msetnx
list command: blpop,brpop,brpoplpush
scripting command: script exists,script flush,script kill,script load
pub/sub command:(全部不支持)psubscribe,publish,punsubscribe,subscribe,unsubscribe
```

<h3 id="第四节">压测</h3>

感谢 redis 提供的 redis-benchmark 工具，用它来做压测挺好的。

* n 表示多少个连接
* r 表示多少个 key,
* t 代表命令

```
zj@zheng-ji.info:~$ redis-benchmark -p 6700 -t smembers,hexists,get,hget,lrange,ltrim,zcard,setex,sadd -n 1000000 -r 100000000

====== GET ======
1000000 requests completed in 12.95 seconds
50 parallel clients
3 bytes payload
keep alive: 1

99.19% <= 1 milliseconds
99.93% <= 2 milliseconds
100.00% <= 2 milliseconds
77220.08 requests per second

====== SADD ======
1000000 requests completed in 10.74 seconds
50 parallel clients
3 bytes payload
keep alive: 1

99.88% <= 1 milliseconds
99.95% <= 2 milliseconds
99.97% <= 3 milliseconds
99.99% <= 4 milliseconds
100.00% <= 4 milliseconds
93144.56 requests per second
```

如作者所言, 性能几乎可以跟直连redis比拟，背后的数据也很均匀,使用twemproxy 观察连接数, 一直都保持在个位数左右。


<h3 id="第五节">摘自极光博客的评论</h3>

* 前端使用 Twemproxy 做代理，后端的 Redis 数据能基本上根据 key 来进行比较均衡的分布。
* 后端一台 Redis 挂掉后，Twemproxy 能够自动摘除。恢复后，Twemproxy 能够自动识别、恢复并重新加入到 Redis 组中重新使用。
* Redis 挂掉后，后端数据是否丢失依据 Redis 本身的策略配置，与 Twemproxy 基本无关。
* 如果要新增加一台 Redis，Twemproxy 需要重启才能生效；并且数据不会自动重新 Reblance，需要人工单独写脚本来实现。
* 如同时部署多个 Twemproxy，配置文件一致（测试配置为distribution：ketama,modula），则可以从任意一个读取，都可以正确读取 key对应的值。
* 多台 Twemproxy 配置一样，客户端分别连接多台 Twemproxy可以在一定条件下提高性能。根据 Server 数量，提高比例在 110-150%之间。
* 如原来已经有 2 个节点 Redis，后续有增加 2 个 Redis，则数据分布计算与原来的 Redis 分布无关，现有数据如果需要分布均匀的话，需要人工单独处理。
* 如果 Twemproxy 的后端节点数量发生变化，Twemproxy 相同算法的前提下，原来的数据必须重新处理分布，否则会存在找不到key值的情况。

------


参考链接

[极光推送的博客](http://blog.jpush.cn/redis-twemproxy-benchmark/)
