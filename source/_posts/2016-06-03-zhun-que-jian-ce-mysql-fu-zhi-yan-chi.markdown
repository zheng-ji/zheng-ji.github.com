---
layout: post
title: "准确监控 MySQL 复制延迟"
date: 2016-06-03 23:35
comments: true
categories: DataBase
description: mysql pt-heartbeat
---

MySQL 建立主从复制后，在 `Slave_IO_Running`,`Slave_SQL_Runing` 都是 Yes 的前提下，通过监控 `Second_Behind_Master` 的数值来判断主从延迟时间，该值为0时是否意味着主从同步是无延迟的呢？

```sql
mysql> show slave status\G;
*************************** 1. row ***************************
Slave_IO_State: Waiting for master to send event
....
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
Seconds_Behind_Master: 0
...
```

很遗憾，我们并不能这样去判断，因为你看到的有可能是假象。


MySQL的同步是异步完成的，其中

* IO thread 接收从主库的 binlog，然后在从库生成 relay log
* SQL thead 解析 relay log 后在从库上进行重放

`Second_Behind_Master`(以下简称SBM) 是 SQL thread 在执行IO thread 生成的relay log的时间差。relay log中event的时间戳是主库上的时间戳，而SQL thread的时间戳是从库上的，SBM 代表的是从库延后主库的时间差。

主库上执行了一个大的操作，这个操作在主库上没执行完毕的时候，从库的 SBM 会显示为0，而当主库执行完毕传到从库上开始执行的时候,SBM 就会显示很大，在网络状况不好的情况下，更是容易出现 SBM 在零和一个巨大的数值反复飘忽的现象。


### pt-heartbeat 帮我们准确地检测

pt-heartbeat 是 percona-toolkit 中用来检测主从延迟的工具，需要在主库和从库同时配合才能完成

* 首先在主库创建监控的表，并定时更新

```
//创建 heartbeat 表
pt-heartbeat --user=root --ask-pass \
            --host=localhost -D <YourDatabase> \
            --create-table --update 

//每隔60s,定时更新状态，以守护进程的方式执行
pt-heartbeat --user=root --ask-pass \
           --host=localhost -D <YourDatabase>\
           --interval=60 --update --replace --daemonize
```
它会在指定的数据库里生产一张名为 heartbeat 的表，每隔60秒定时更新binlog 文件和位置，以及时间戳。

```
+----------------------------+-----------+------------------+-----------+-----------------------+---------------------+
| ts                         | server_id | file             | position  | relay_master_log_file | exec_master_log_pos |
+----------------------------+-----------+------------------+-----------+-----------------------+---------------------+
| 2016-06-03T22:26:29.000720 |         6 | mysql-bin.004| 716| mysql-bin.002|           291330290 |
+----------------------------+-----------+------------------+-----------+-----------------------+---------------------+
```

* 接着在从库以守护进程执行定期检测,并将结果重定向到文本

```
pt-heartbeat --user=root --ask-pass \
     --host=localhost -D <YourDatabase> --interval=60 \
     --file=/tmp/output.txt --monitor --daemonize
```

文本的内容只有一行，每隔指定的时间就会被覆盖

```
29.00s [ 30.20s,  6.04s,  2.01s ]
```

29s 表示的是瞬间的延迟时间，30.20s 表示1分钟的延迟时间，6.04秒表示5分钟的延迟时间，2.01秒表示以及15分钟的延迟时间，在主从机器时间校准的前提下，这个数据才是客观准确的主从延迟。
