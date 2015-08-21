---
layout: post
title: "supervisor监听器"
date: 2015-08-21 16:36
comments: true
categories: Server
categories: 
description: supervisor 监控 
---

我们之前使用 monit 来监控程序状态的，服务比较多是用 supervisor 启动的，如果我们能通过监测 supervisor 事件变化来做监控，就可以写一套通用的监控程序。

庆幸的是，supervisor 的 `eventListener` 支持我的设想。

这个监控程序需要用 supervisor 启动，类型不再是`program`, 而是`eventlistener`，这里有几个比较耗时的地方需要记录下。

* supervisor 有独特的通信协议,需要遵循，否则通讯不会被触发

```
def write_stdout(self, s):
    sys.stdout.write(s)
    sys.stdout.flush()

write_stdout('READY\n') //类似开始握手
write_stdout('RESULT 2\nOK') //结束通讯
```

* 需要从标准输入端读取事件,而且他是个阻塞的事件模型

```
while 1:
    self.write_stdout('READY\n')
    line = sys.stdin.readline()
    do_some_thing()
    self.write_stdout('RESULT 2\nOK')
```

* supervisor 配置文件需要订阅事件

```
[eventlistener:alarm]
user=zj
command=/usr/bin/python /home/ymserver/bin/alarm/main.py
events=PROCESS_STATE_EXITED,PROCESS_STATE_STOPPED,PROCESS_STATE_FATAL

# 记录控制台输出的日志位置
stderr_logfile=/home/zj/log/supervisor/alarm.err.log
stdout_logfile=/home/zj/log/supervisor/alarm.output.log
```

弄好 supervisor 配置，以及部署好代码之后，需要重启 supervisor 才会真正的订阅事件。
从此 supervisor 管理的程序一旦有 `FATAL`,`EXIT` 等状态就会触发程序，程序中就会触发自定义的报警。

----


[代码](https://github.com/zheng-ji/ToyCollection/tree/master/supervisor-listener)

Happy Hack!

