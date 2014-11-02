---
layout: post
title: "唯一索引引发的思考"
date: 2014-10-23 23:18
comments: true
keywords: MySQL Index
description: MySQL Index
categories: DataBase
---

最近需要改动线上一个有千万条记录的表，涉及到加字段操作，这个表有索引，按照经验，为了加速修改表结构,去掉索引。由于我删除的是Unique Index, 而服务一直在写,程序依赖数据库的唯一索引去重,导致瞬间有重复数据,唯一索引重新加上时会报 `Duplicated Key entry Error`,

###回放事件###


我们的数据表，之前是有 `UNIQUE INDEX(cuid, aid)`, 因为去掉索引，服务持续写入，导致有重复记录，所幸的是，这是一个统计表, 不影响功能，所以需要找出重复的记录

```
select cuid,aid from (
    select cuid, aid,count(1) as num
    from register_chn 
    group by cuid,aid having num > 1
) t;
```

显示有8条记录,如果手动删除,是很慢且愚蠢的做法,还是用 SQL 执行,镇定之后执行

```
delete from register_chn where (cuid,aid) in (
    select cuid,aid from (
        select cuid, aid,count(1) as num
        from register_chn
        group by cuid,aid having num > 1
    ) t
);
```
影响了８条记录.然后瞬间加上索引.所幸是成功了, 事实上当时的合理操作应该是用事务。

```
BEGIN;
delete from register_chn where (cuid,aid) in (
    select cuid,aid from (
        select cuid, aid,count(1) as num
        from register_chn
        group by cuid,aid having num > 1
    ) t
);
alter table register_chn add unique index(cuid, aid);
COMMINT;
```

###引发的思考###

后来想想, 上述方法虽然解决问题了, 但是有点碰运气成分。如果频繁快速地产生重复记录,也许就没那么好运了,事实上可以执行以下 SQL:

```
alter ignore table register_chn add unique index(cuid, aid)；
```

如果你以为很简单,那就错了。这个方法在 MySQL 5.0 上使用是没问题的，但是在5.6 之前是有bug的，亲自测试Percona 版本5.5, 的确会失败
官方的解决方法是：

```
set session old_alter_table = on;
alter ignore table register_chn add unique index(cuid, aid);
```

在有主备的情况,记得执行前设置一下

```
set session sql_log_bin=off. 
```
以免备库报错。同样还需要在备库重复一下主库的操作, 这也算是一个不太完美的解决思路。


