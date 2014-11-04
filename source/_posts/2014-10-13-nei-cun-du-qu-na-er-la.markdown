---
layout: post
title: "内存都去哪儿啦"
date: 2014-10-13 21:20
comments: true
keywords: 内存 Top
description: 内存
categories: System
---

###看到的现象###

今天在游戏服务器上使用了top 命令系统使用参数，发现内存16G 的内存用了12G, 如下
{% img /images/2014/10/top.png %}
但是同时在线人数不多，机器的负载并不是很大，并发也不高, 只有900+。
{% img /images/2014/10/nestat.png %}
而游戏的服务端代码仅仅使用了7.8G, 我们不禁会问，内存去哪里了？难道是系统自己占用的内存呢，不可能一个8核16G的机器系统自己占用了4G吧?

带着这个疑问，查阅了资料，发现其中并非表明所看到的这么简单. 我们来看以下 使用 `free -m` 命令
{% img /images/2014/10/free.png %}

### Mem 参数 解释### 
+ total 内存总数: 15875 , total = used + free
+ used 已经使用的内存数: 11900
+ free 空闲的内存数: 3975
+ shared 当前已经废弃不用，总是0
+ buffers: Buffer Cache内存数: 145
+ cached: Page Cache内存数: 5561

### -/+ buffers/cache的解惑 ####
+ -buffers/cache 的内存数: 6193 (大致等于第1行的 used - buffers - cached), 反映的是被程序实实在在吃掉的内存，
+ +buffers/cache 的内存数: 9682 (大致等于第1行的 free + buffers + cached), 反映的是可以挪用的内存总数

### 两种cache ###
因为cpu速度明显快过内存， 为了提高磁盘存取效率, Linux做了一些精心的设计, 采取了两种主要Cache方式, 来做速度的过度, 这些Cache有效缩短了 I/O系统调用(如read,write,getdents)的时间。

+ Buffer Cache, 针对磁盘块的读写
+ Page Cache。针对文件inode的读写。

### 内存解读的区别 ###

第1行`(mem)的used/free`与第2行`(-/+ buffers/cache) used/free`的区别在于角度的不同:

+ 第一行因为对于OS，buffers/cached 都是属于被使用，所以他的可用内存是3975M,已用内存是11900MB,其中包括,内核（OS）使用 + 应用使用的+ buffers + cached.
+ 第2行所指的是从应用程序角度来看，对于应用程序来说，buffers/cached 是可用的，是为了提高文件读取的性能而设，当应用程序要用到内存的时候，buffer/cached会很快地被回收。所以从应用程序的角度来说，

```
可用内存=系统free memory + buffers + cached.
```
所以，回到话题开头，虽然内存显示用了12.2G，正在被应用程序使用的是7.9G 还有5.6(cache+buffer) +  4G free 可用.内存原来在这里！

获益匪浅 :) 
