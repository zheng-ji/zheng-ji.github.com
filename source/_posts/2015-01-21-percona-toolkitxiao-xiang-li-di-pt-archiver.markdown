---
layout: post
title: "Percona 小箱里的pt-archiver"
date: 2015-01-21 23:27
comments: true
keywords: MySQL pt-archiver
categories: DataBase
---

在管理线上数据库，时常要做一些数据归档操作，在没有了解 `Percona toolkit` 之前，第一个想到的是在夜深人静的时候使用 `MySqlDump` 来完成这件事情。但它不是我们的优质选择，理由有：

+ `MySqlDump` 只能备份在本机，不能直接做远端备份
+  导出数据量太大的时候会锁表, 即使它的速度很快，但是在线上服务这是很危险的操作
+  它仅仅只能导出，无法做到同时删除(可能不是太有必要)

面对上述的场景，`Percona Toolkit` 让DBA 有了更好地选择，[pt-archiver](http://www.percona.com/doc/percona-toolkit/2.1/pt-archiver.html) 应时而生。

## pt-archiver 介绍：

根据官方文档的说法，几乎不会对线上的OTLP操作有影响：

> The goal is a low-impact, forward-only job to nibble old data out of the table without impacting OLTP queries much

它可以帮助我们将数据归档到文件, 另一个数据库，或者同一个数据库的另一个表, 亦或是用于合并两个表的内容。

## 用法介绍：

```
pt-archiver [OPTION...] --source DSN --where WHERE
```

归档的文件方便使用 load data infile 命令导入数据。另外你还可以用它来执行 delete 操作。这个工具默认的会删除源中的数据，使用的时候请注意。

假如我们将数据库里符合条件的记录归档到文件，并不做删除操作。

```
pt-archiver --ask-pass --progress 100000 --no-delete --no-check-charset --source h=localhost,u=root,D=blog,t=comment--file /home/ubuntu/tmp/comment--where 'time < "2013-12-31h"'
```

注意的选项参数：

```
--ask-pass        提示要求输密码
--progress num    执行num行在界面通知我们
--source h,D,t    数据源
--no-delete       加上这个参数并不会执行删除操作
--dry-run         仅仅将执行语句打印在终端，事实上并不执行。可以用于检测执行过程
--where           执行语句，需要用冒号包围起来
--limit           批量操作的数量，合理提高这个数值可以加快archive速度
```

### 小总结

  通过开启 mysql 的 `general log`, 可以发现pt-archiver 执行时，是分批commit 的事务，因此执行效率会慢，在8核16G 内存的生产环境机器备份 1kw 条记录, 耗时150 分钟。 但基本不对服务造成影响，而且可以不用深夜进行, 值得一用。

  最近攒了好多好工具和经验，要好好整理搬上来和大家分享才是。
