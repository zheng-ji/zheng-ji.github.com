---
layout: post
title: "Percona Server 做主从"
date: 2014-09-07 15:31
comments: true
description: Mysql 主从
categories: DataBase
---

数据库做主从目的:

+ 故障恢复， 柔性可用
+ 也可以做读写分离

实验过程中 mysql 用的版本是 Percona Server ，

[安装过程](http://www.percona.com/doc/percona-server/5.5/installation/apt_repo.html)

由于修改默认的数据目录，数据文件不再使用 `/var/lib/mysql`,数据文件夹被我安放位置是 `/data/mysql/data` 同时log 目录也放在这里 `/data/mysql/log` 
注意需要修改目录属主

```
sudo chown mysql:mysql  -R /data/mysql
然后执行 sudo mysql_install_db 
```


这时候会发生 `sudo service mysql stop` 失败，原因和方法见此 [神贴](http://serverfault.com/questions/32692/cant-start-stop-mysql-service/420222#420222)

好了开始进入正题了，备份数据的原理

+ 在主库上把数据更改记录到二进制日志 (Binary Log) 
+ 备库将主库的日志复制到自己的中继日志中
+ 备库读取中继日志的事件，将其重放到备库的数据之上


从其他服务器克隆备库的方法：

+ 使用冷备份：关闭主库，将数据复制到备库，重启主库后，会使用一个新的二进制日志文件，在备库执行 CHANGE MASTER TO 指向这个文件的起始处，缺陷在于关闭主库
+ 使用热备份，如果仅使用，mysqlhotcopy,或rsync来复制数据
+ 使用快照或备份：需要知道二进制日志坐标，就可以使用主库的快照和备份来初始化备库，只需要把备份或快照恢复到备库，然后使用 CHANGE MASTER TO 指定二进制日志的坐标.
+ 用`Percona Xtrabackup`  个人推荐  [链接](http://www.percona.com/doc/percona-xtrabackup/2.1/howtos/setting_up_replication.html)

----

### 实际的步骤如下

+ 备份主库的数据

```
innobackupex --user=root --password=xxx /home/zj/backup
```
在/home/zj/backup目录下就生成了2014-08-21_10-11-4` 目录

+ 复制数据到从库，通过`scp`将上一步生成的目录放置在从库机器(~/tmp`)将原来的data目录备份, 在从库机器执行

```
mv /data/mysql/data /data/mysql/data_bak
mv ~/tmp/2014-08-21_10-11-46  /data/mysql/data
chown mysql:mysql -R /data/mysql/data
```

+ 配置主服务器

```
GRANT REPLICATION SLAVE ON *.*  TO 'repl'@'$slaveip' IDENTIFIED BY ''slavepass
```

+ 配置从数据库,并重启

```
server-id=2
```

+ 开始复制

需要定位位置

```
cat /data/mysql/data/xtrabackup_binlog_info
log-bin.000001     481
```

+ 在从库执行 

```
mysql> CHANGE MASTER TO MASTER_HOST='$masterip',
       MASTER_USER='repl',
       MASTER_PASSWORD='$slavepass',
       MASTER_LOG_FILE='log-bin.000001',
       MASTER_LOG_POS=481;
mysql> START SLAVE;
mysql> SHOW SLAVE STATUS \G
 ```

-----

系统学习的书籍 

[《高性能 mysql》](http://book.douban.com/subject/23008813/)
 
