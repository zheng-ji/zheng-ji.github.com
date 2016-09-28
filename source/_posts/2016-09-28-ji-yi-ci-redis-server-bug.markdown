---
layout: post
title: "记一次使用 redis-server 诡异的bug"
date: 2016-09-28 08:21
description: Redis 协议 
categories: Programe
---

记录昨天定位一个诡异 bug 的过程，我耗费了不少精力，你若有兴趣，请带着一点耐心看完它。

我们使用 [go-redis-server](https://github.com/youmi/go-redis-server) 开发具有 redis 接口的服务。 按照文档，我们实现了如下接口，其背后访问的是 AWS 的 Dynamodb, 我们的服务也开发了监控接口，以供我们这些程序狗知道它发生了什么。

```
func (handler *RedisHandler) Get(key string) (result string, err error)
func (handler *RedisHandler) Set(key string, val string) (err error)
func (handler *RedisHandler) Del(key string) (count int, err error)
```

这样，我们就能通过redis客户端来执行 Get，Set，Del 操作。

### 问题背景

我在批量写入几千条数据时，通过监控接口，我看到服务突然卡住的样子，没有 Get，Set的统计信息了。但我能肯定的是客户端一直有数据在往服务写入，或者读写。
同时从 AWS 监控得到的反馈是 Dynamodb 使用超过预设值，我调高了读写预设值，重启服务，就恢复可用（重启大法好）。
程序试运行十多天，只发生过一次异常，之后再无重现。
这个事情没有搞清楚，就成为我一个心结。


### 重现问题

我们来看看代码

```
func (handler *RedisHandler) 
Set(key string, val string) (err error) {
	...

	m, err := JSONToMap(val)
	_, err = table.PutItem(m)
	if err != nil {
		log.Log.Errorf("PutItem failed, 
		table: %s, primary key: %s, value: %+v, 
		err: %s", 
		tableName, primaryVal, m, err.Error())
		return
	}
	return
}
```

Set接口，简单的将数据写入dynamodb，dynamodb 如果异常就返回错误，然后通过redis协议返回给客户端。

我很大程度确定那时候是因为 AWS Dynamodb 的异常导致这个错误的。除非这个巧合太牛逼了，难道 go-redis-server 接口不支持返回错误不成吗？这个猜想很快就被我们用实验否定了。

我想重新触发 AWS Dynamodb 返回写容量超标的错误，测试了一阵子，并不容易重现。有点沮丧，这个时候我回忆起 aws-go-sdk的特点，如果给dynamodb 字段 赋予空值，是会有异常返回的，
类似如下：

```
ValidationException: One or more parameter values were invalid:
An AttributeValue may not contain an empty string
	status code: 400
```

空值的测试明显容易制造。我在本地也开启服务，端口是1234，用pyredis 作为客户端做测试


测试脚本

```
import redis
import json
import random
r = redis.Redis(host='localhost',port=1234,db=0)

table_name = 'test'
primary_key = 'a'

i = 0
while True:
    if i > 2:
        break
    data = {
        'a': str(random.random()),
        'b': ‘’,
    }
    key = '{0}:{1}:{2}'.format(table_name, primary_key, data[primary_key])
    value = json.dumps(data)
    r.set(key, value)
    i = i + 1
```

发现测试程序卡在终端，一动不动，strace 测试程序

```
$ sudo strace -p 22404
Process 22404 attached
recvfrom(3,
```

感觉像是在等待服务器返回，但是等不到回报的样子。

### 试着解决问题

难道 go-redis-server 这个框架有猫腻，我就开始了一下午的看源码之旅。不得不说源码写的真好。回头看我们出错的代码片段，我试着修改 err信息，修改成自己定义的错误字符串。

```
func (handler *RedisHandler) Set(key string, val string) (err error) {
	...

	m, err := JSONToMap(val)
	_, err = table.PutItem(m)
	if err != nil {
		err = errors.New("I am a Custom Error")
		return
	}
	return
}
```

再次执行测试脚本，这一次。2条测试数据很快就执行结束，并无阻塞。
好像看到了一丝曙光：error 内容的不一样，导致不一样的结果。类型一样，那么我能怀疑的就是格式，或者长度了。

```
AWS 错误信息的格式：
ValidationException: One or more parameter values were invalid:
An AttributeValue may not contain an empty string
	status code: 400

我自定义错误信息的格式：
I am a Custom Error
```

我特意加长了自定义错误的长度，结果也是能顺利执行不阻塞客户端。但是我给自定义错误字符串加入了换行符，果然客户端再次测试会出现阻塞。当出现错误的时候，源码中是调用了 ErrorReply.WriteTo 这个函数，特别的。返回错误的协议格式是

>第一个字节将是“-”,并以CR + LF 结尾

配合源码，以下是 go-redis-server 最核心的逻辑调度代码。


```
for {
	request, err := parseRequest(conn)
	if err != nil {
		return err
	}
		request.Host = clientAddr
		request.ClientChan = clientChan
		reply, err := srv.Apply(request)
		if err != nil {
			return err
		}
		if _, err = reply.WriteTo(conn); err != nil {
			return err
		}
	}
	return nil
}
func (er *ErrorReply) WriteTo(w io.Writer) (int64, error) {
	n, err := w.Write([]byte("-" + er.code + " " + er.message + "\r\n"))
	return int64(n), err
}
```

从 [Redis协议官方文档](http://redis.cn/topics/protocol.html)，可以确定客户端与服务器端之间传输的每个 Redis 命令或者数据都以 `\r\n` 结尾。 我们的错误信息中间杀出了 `\n`，导致协议错乱，redis 客户端不能理解协议，就阻塞了。


### 一点感悟

* 以后我们使用 go-redis-server 的服务时候，要记得检查返回的字符串或者错误信息有没有包含换行符，如果有，最好做一次过滤替换。

* 出现这个bug,我和同事都觉得不可思议，非常神奇。在没有其他直观线索的条件下，阅读使用的库的源码，并在源码加上一些输出验证语言加以辅助，收到了效果，的确需要一些耐心。但我觉得是值得的。

* 李笑来说有效阅读就是精度，这次阅读代码过程中我还有意外的收获，我发现了reflect的妙用，以及函数注册在框架的用法，读完觉得很满足开心的样子，值得再记录一番。

---

附链接：

- [Redis官方协议](http://redis.cn/topics/protocol.html)
- [go-redis-server](https://github.com/youmi/go-redis-server)

