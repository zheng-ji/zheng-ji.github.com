---
layout: post
title: "分布式处理框架-Gearman"
date: 2013-06-20 00:00
comments: true
categories: Server 
---
近日折腾Gearman.有兴趣可以参考阅读[这个](http://www.gearman.org/),[这个](http://pythonhosted.org/gearman/)，还有[这个](http://www.s135.com/dips/)

官方说法：

>Gearman provides a generic application framework to farm out work to other machines or processes that are better suited to do the work. It allows you to do work in parallel, to load balance processing, and to call functions between languages. It can be used in a variety of applications, from high-availability web sites to the transport of database replication events. In other words, it is the nervous system for how distributed processing communicates. 

Gearman是一个提供机器与进程之间相互协作的通信框架。可以让你并行处理作业以及负载均衡，提供不同的语言接口使得不同语言可以互相协作。用处很广泛，从web站点到数据库Replication事件。换句话说，是一个分布式的通讯框架

#### 特点

+ 开源：
    多语言支持：Gearman支持的语言种类非常丰富。让我们能够用一种语言来编写Worker程序，但是用另外一种语言编写Client程序。

+ 灵活：不必拘泥于固定的形式。您可以采用你希望的任何形式，例如 Map/Reduce。

+ 快速：Gearman的协议非常简单，并且有一个用C语言实现的，经过优化的服务器，保证应用的负载在非常低的水平。

+ 可植入：因为Gearman非常小巧、灵活置入到现有的任何系统中。

+ 无单点故障：Gearman可扩展系统，可以避免系统的失败。

#### 场景
听上去很诱惑的样子了。试想下：
我们的服务器有IO读写密集型，存储密集型，CPU使用不频繁，内存使用不频繁。如果我们能根据业务充分调动起各个服务器的特点。实现服务器的最大价值。Gearman可以登上舞台 ：）

图解工作流
{% img /images/2013/08/gearman.png %}

#### Gearman 有三个角色
+ Job Server:中央调度，消息中心，负载均衡
+ Client：发起Job,可由任何语言编写
+ Worker：负责执行Job，执行业务流

实操安装Gearman（以ubuntu为例子）,默认的端口4730

```
sudo apt-get install gearman
```

安装python-gearman

```
sudo apt-get install python-gearman
```

用python编写
worker代码

```
#worker.py
import os
import gearman
import math 
 
class CustomGearmanWorker(gearman.GearmanWorker):
   def on_job_execute(self, current_job):
          return super(CustomGearmanWorker, self).on_job_execute(current_job)
def task_callback(gearman_worker, job):
    print job.data
    return job.data + " has received" 
                   
new_worker = CustomGearmanWorker(['127.0.0.1:4730'])
new_worker.register_task("ping", task_callback)
new_worker.work()
```

client代码

```python
#!/usr/bin/env python2.7
# -*- coding: utf-8 -*-
# # file: client.py
from gearman import GearmanClient
new_client = GearmanClient(['127.0.0.1:4730'])
current_request = new_client.submit_job('ping', 'foo')
new_result = current_request.result
print new_resul
```

结果

```
foo has received
```

用Go测试下

```
#gearman的Go库,内有example
go get https://github.com/mikespook/gearman-go
```

Happy Hacking!
