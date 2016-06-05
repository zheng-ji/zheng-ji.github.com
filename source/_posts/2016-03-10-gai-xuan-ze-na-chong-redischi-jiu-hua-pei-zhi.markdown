---
layout: post
title: "Redis 该选择哪种持久化配置"
date: 2016-03-10 23:32
comments: true
categories: System
description: Redsi AOF RDB
---

这个标题或许会让你想起[《黑客帝国》](https://movie.douban.com/subject/1291843/)里经典的台词，你要选择蓝色药丸，还是红色药丸？

Redis 是我们重度使用的一个开源软件，对它的持久化配置做一番相对深入的总结，是值得的。目前它有两种主流的持久化存储方式 SnapShot 以及 AOF 。

* [什么是 Snapshot](#第一节)
* [什么是 AOF ](#第二节)
* [选择哪种药丸](#第三节)


<h3 id="第一节">什么是 Snapshot</h3>

Snapshot 将内存中数据以结构化的方式序列化到 rdb 文件中，是默认的持久化方式，便于解析引擎快速解析和内存实施。快照得由间隔时间，变更次数同时符合才会触发， 该过程中并不阻塞客户端请求，copy-on-write 方式也意味着极端情况下可能会导致实际数据2倍内存的使用量。它首先将数据写入临时文件，结束后，将临时文件重名为 dump.rdb。可以使用 `redis-check-dump` 用来检测完整性

只有快照结束后才会将旧的文件替换成新的，因此任何时候 RDB 文件都是完整的。如果在触发 snapshot 之前，server 失效。会导致上一个时间点之后的数据未能序列化到 rdb 文件，安全性上稍弱。 

我们可手动执行 save 或 bgsave 命令让 redis 执行快照。两个命令的区别在于:

* save 是由主进程进行快照操作，会阻塞其它请求;
* bgsave 会通过 fork 子进程进行快照操作;

RDB 文件默认是经过压缩的二进制文件，占用的空间会小于内存中的数据，更加利于传输。设置如下，可以关闭快照功能

```
save ""
```


#### 相关配置

```
# snapshot触发的时机，save <seconds> <changes>， 比如600秒有2个操作
save 600 2
# 当snapshot 时出现错误无法继续时，是否阻塞客户端变更操作 
stop-writes-on-bgsave-error yes 
# 是否启用rdb文件压缩，默认为 yes cpu消耗，快速传输  
rdbcompression yes  
# rdb文件名称  
dbfilename dump.rdb  
```


<h3 id="第二节">什么是 AOF</h3>

Append-only file，将 `操作 + 数据` 以格式化指令的方式追加到操作日志文件的尾部，在 append 操作返回后， 已经写入到文件或者即将写入，才进行实际的数据变更，日志文件保存了历史的操作过程；当 server 需要数据恢复时，可以直接回放此日志文件，即可还原所有的操作过程。 如果你期望数据更少的丢失，那么可以采用 AOF 模式。可以用 redis-check-aof 检测文件是否完整。

AOF 就是日志会记录变更操(例如：set/del等)，会导致AOF文件非常的庞大，意味着server失效后，数据恢复的过程将会很长；事实上，一条数据经过多次变更，将会产生多条AOF记录，其实只要保存当前的状态，历史的操作记录是可以抛弃的， 由此催生了 AOF ReWrite。

#### 什么是 AOF Rewrite

其实是压缩 AOF 文件的过程，Redis 采取了类似 Snapshot 的方式：基于 `copy-on-write`，全量遍历内存中数据，然后逐个序列到 aof 文件中。因此 AOF Rewrite 能够正确反应当前内存数据的状态， Rewrite 过程中，新的变更操作将仍然被写入到原 AOF 文件中，同时这些新的变更操作也会被收集起来， 并不阻塞客户端请求。


#### 相关配置

```
##只有在yes下，aof重写/文件同步等特性才会生效  
appendonly no  
  
##指定aof文件名称  
appendfilename appendonly.aof  
  
##指定aof操作中文件同步策略，有三个合法值：always everysec no，默认为everysec  
appendfsync everysec  

##在aof-rewrite期间，appendfsync 是否暂缓文件同步，no 表示不暂缓，yes 表示暂缓，默认为no  
no-appendfsync-on-rewrite no  
  
##aof文件rewrite触发的最小文件尺寸 只有大于此aof文件大于此尺寸是才会触发rewrite，默认64mb，建议512mb  
auto-aof-rewrite-min-size 64mb  
  
##相对于上一次rewrite，本次rewrite触发时aof文件应该增长的百分比
auto-aof-rewrite-percentage 100  
```

#### appendfsync 方式：

* always：每一条 aof 记录都立即同步到文件，这是最安全的方式，但是更多的磁盘操作和阻塞延迟，IO 开支较大。
* everysec：每秒同步一次，性能和安全也是redis推荐的方式。如果服务器故障，有可能导致最近一秒内aof记录丢失。
* no：redis并不直接调用文件同步，而是交给操作系统来处理，操作系统可以根据buffer填充情况等择机触发同步；性能较好，在物理服务器故障时，数据丢失量会因OS配置有关。


<h3 id="第三节">选择哪种药丸</h3>

* AOF更安全，可将数据及时同步到文件中，但需要较多的磁盘IO，AOF文件尺寸较大，文件内容恢复相对较慢， 也更完整。
* Snapshot，安全性较差，它是正常时期数据备份及 master-slave 数据同步的最佳手段，文件尺寸较小，恢复数度较快。

#### 主从架构的环境下的选择

* 通常 master 使用AOF，slave 使用 Snapshot，master 需要确保数据完整性，slave 提供只读服务.
* 如果你的网络稳定性差， 物理环境糟糕情况下，那么 master， slave均采取 AOF，这个在 master， slave角色切换时，可以减少时间成本；
