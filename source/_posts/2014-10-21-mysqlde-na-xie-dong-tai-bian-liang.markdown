---
layout: post
title: "mysql的那些动态变量"
date: 2014-10-21 23:18
comments: true
categories: DB
---

我想写一篇长期更新的文章，随着知识积累翻新的文章，而最近遇到的Mysql 动态变量，便是一个可以琢磨的点，变量比较多，并且需要在每次采坑才来的深刻。

* tmp_table_size
这个是临时表的大小，与之对应的有 `max_heap_size` 单位是Byte
`tmp_table_size` 的大小关乎临时表的大小, 如果设置太小，会导致一些数据丢失

```
set global tmp_table_size = 512 * 1024 *1024 
```

* max_connections
当`show processlist` 发现有好多链接的时候，客户端想登陆mysql却失败。
可以执行, 看看是不是现有的链接数已经逼近

```
show global variable like '%max%'
set global max_connections 1000
```






