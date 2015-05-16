---
layout: post
title: "MySQL Slave Relay log Corrupt 恢复"
date: 2015-05-10 20:02
comments: true
categories: DataBase
description: mys1l slave relay log
---

### 现象 ### 
周日早晨收到 `ganglia` 报警, 内容是：

```
MySQL_Slave_SQL is 0.00
```
意味着从库同步有问题了。这时候进入从库看看状态

```
show slave status\G;
```

看到

```
Slave_IO_State: Waiting for master to send event
   Master_Host: xxx.xxx.xxx
   Master_User: replication
   Master_Port: 3306
   Connect_Retry: 60
   Master_Log_File: mysql-bin.000028
   ad_Master_Log_Pos: 982714864
   Relay_Log_File: relay-bin.000143
   Relay_Master_Log_File: mysql-bin.000028
   Slave_IO_Running: Yes
   Slave_SQL_Running: No
   Replicate_Do_DB:
 Replicate_Ignore_DB:
 Replicate_Do_Table:
 Replicate_Ignore_Table:
 Replicate_Wild_Do_Table:
 Replicate_Wild_Ignore_Table:
 Last_Errno: 1594
 Last_Error: Relay log read failure: Could not parse relay log event entry. The possible reasons are: the master's binary log is corrupted (you can check this by running 'mysqlbinlog' on the binary log), the slave's relay log is corrupted (you can check this by running 'mysqlbinlog' on the relay log), a network problem, or a bug in the master's or slave's MySQL code. If you want to check the master's binary log or slave's relay log, you will be able to know their names by issuing 'SHOW SLAVE STATUS' on this slave.
 Skip_Counter: 3
 Exec_Master_Log_Pos: 974999870
 Relay_Log_Space: 399910514
 Until_Condition: None
```
 
是因为从库的 relay log 损坏导致的从库停止执行同步, 后面从 MySQL 的错误日志发现是由几个没经过优化的大查询导致从库内存使用较大，mysqld_safe 被迫重启了。

### 解决方法 ###

找到：

```
Relay_Master_Log_File: mysql-bin.000028
Exec_Master_Log_Pos: 974999870
```

执行

```
mysql> stop slave ;
mysql> change master to master_log_file='mysql-bin.000028',master_log_pos=974999870;
mysql> start slave;
```
                                                                                  
这样就重新追上了。以后还是要提高周围同事使用 MySQL 的优化意识啊。
