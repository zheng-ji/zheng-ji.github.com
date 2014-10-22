---
layout: post
title: "mysql的那些动态变量"
date: 2014-10-21 23:18
comments: true
categories: DB
---

我想写一篇长期更新的文章，随着知识积累持续更新的文章，而最近遇到的在涉及的Mysql优化，便是一个可以琢磨的点，变量比较多，并且需要在每次采坑才来的深刻。

* tmp_table_size
这个是临时表的大小，与之对应的有 `max_heap_size` 单位是Byte, `tmp_table_size` 的大小关乎临时表的大小, 如果设置太小，会导致一些数据直接写入磁盘，比较蛋疼, 看了官方文档，涉及到`Dynamic Variable=Yes`
那么这种配置便可以直接在终端用语句设置，并生效，不需要重启机器

```
set global tmp_table_size = 512 * 1024 *1024 
```

* max_connections
当`show processlist` 发现有好多链接的时候，客户端想登陆mysql却失败。可以执行, 看看是不是现有的链接数已经逼近

```
show global variable like '%max%'
set global max_connections 1000
```
* thread_concurrency
同一时间与性的线程设置，如果分配不当，会导致`mysql`不能充分使用多｀cpu｀, 出现同一时刻只有一个核在工作 
`thread_concurrency`应设为`CPU`核数的2倍. 比如有一个双核的CPU, 那么`thread_concurrency`的应该为4; 2个双核的cpu, `thread_concurrency`的值应为8.

* max_allowed_packet
如果需要进行mysqldump这类操作，需要调大
```
set global max_allowed_packet = 2*1024*1024*10
```
