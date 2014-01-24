---
layout: post
title: "ZeroRPC-基于ZMQ的RPC通讯库"
date: 2013-09-29 20:56
comments: true
categories: Server NetWork
---
[dotCloud](https://www.dotcloud.com/) 是一家具有伟大基因的公司,我认为的伟大是有着开源贡献的情怀，就像Amazon,Google等，而不是国内的某些巨头，虽然作为新兴的云服务提供商还不足以比肩巨头，致力于用技术解决公司的运营问题的同时也回馈社会，让我为之喝彩.我也有理由相信这样的公司会走的更远，因为他们有胸怀有远见，是否重视技术就更不言而喻了.

这篇文章旨在介绍由**dotCloud**开源的[ZeroRpc](https://github.com/dotcloud/zerorpc-python),在dotcloud公司的基础服务得到很大应用，在阅读其manual之后更是被其简洁明了的使用方法所吸引 
>ZeroRPC is a modern communication layer for distributed systems built on top of ZeroMQ,

一直以来我一直喜欢python 的简洁，用过ZeroMQ做过网络通讯方面的编程，也使用RPC 做过远程过程调用，上次使用LevelDB的RPC是用python的[第三方库] (https://github.com/dotcloud/zerorpc-python)

这次刚好在Github上面看到这样的好玩意，便想与大家分享,ZeroRpc不仅仅支持代码层面的调用，也支持CLi， 这种设计本身就很有弹性.赞！

安装zerorpc
```
sudo pip install zerorpc 
```

在还没开始看demo之前
我们需要了解ZeroRpc是由三层架构组成：

* 传输层是使用[ZMQ](http://www.zeromq.org/) 以及msgpack(http://msgpack.org/),基于ZeroMQ的分布式通讯层,通讯的数据被MsgPack 序列化过所以更快
* 消息层,比较复杂,处理heartbeat, multiplexing, and events.
* RPC层:处理请求,响应

官方的[文档](http://zerorpc.dotcloud.com/)给出以下demo

####server.py #####
```
import zerorpc
class HelloRPC(object):
    def hello(self, name):
        return "Hello, %s" % name

s = zerorpc.Server(HelloRPC())
s.bind("tcp://0.0.0.0:4242")
s.run()
```
####client.py
```
import zerorpc

c = zerorpc.Client()
c.connect("tcp://127.0.0.1:4242")
print c.hello("RPC")
```
client也可用命令行代替
```
zerorpc tcp://127.0.0.1:4242 hello RPC
```

够见面易懂了吧
再来一个返回连续字节流的例子
####server.py
```python
import zerorpc

class StreamingRPC(object):
    @zerorpc.stream
    def streaming_range(self, fr, to, step):
        return xrange(fr, to, step)

s = zerorpc.Server(StreamingRPC())
s.bind("tcp://0.0.0.0:4242")
s.run()
```

####client.py
```python
import zerorpc

c = zerorpc.Client()
c.connect("tcp://127.0.0.1:4242")

for item in c.streaming_range(10, 20, 2):
    print item
```

client也可用命令行代替,--json 表示头部是一个json对象

```
zerorpc tcp://127.0.0.1:4242 streaming_range 10 20 2 --json
```

Happy Hacking


