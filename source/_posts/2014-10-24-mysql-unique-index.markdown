---
layout: post
title: "唯一索引引发的思考"
date: 2014-10-23 23:18
comments: true
categories: DB
---

昨晚凌晨1点，需要改动线上一个涉及千万条记录的表，涉及到加表操作，这个表有索引，按照经验，我去掉索引，为了加速修改表结构.屡试不爽.速度的确很快．
但这次问题来了，由于我删除的是Unique Index, 而服务一直在写，程序依赖数据库的唯一索引去重，导致瞬间有重复数据，唯一索引重新加上会报 'Duplicated Key entry Error',
因为有重复数据一直写入，所以导入索引重新添加失败.

###回放事件###


简单介绍下场景，我们的数据表，之前是有 `UNIQUE INDEX(cuid, aid)`, 因为去掉索引，导致有重复记录，所幸的是，这是一个统计表, 不影响功能，我得找出重复的记录

```
select cuid,aid from ( select cuid, aid,count(1) as num from register_chn group by cuid,aid having num > 1) t;
```
显示有8条记录，

想到用手动删除，是很慢而缺愚蠢的做法，还是用 SQL 了事吧，情况紧急，镇定了之后执行

```
delete from register_chn where (cuid,aid) in (
    select cuid,aid from ( select cuid, aid,count(1)as num from register_chn group by cuid,aid having num > 1) t
);
```
影响了８条记录.然后瞬间加索引.所幸是成功了.惊出了冷汗...

事实上当时的合理操作应该是用事务

```
BEGIN;
delete from register_chn where (cuid,aid) in (
    select cuid,aid from ( select cuid, aid,count(1)as num from register_chn group by cuid,aid having num > 1) t
);
alter table register_chn add unique index(cuid, aid);

COMMINT;
```

###引发的思考###

后来想想, 上述方法虽然解决问题了，但是有点碰运气成分，如果频繁快速地产生重复记录，也许就没那么好运了, 觉得肯定有方法的. 果然，可以执行

```
alter ignore table register_chn add unique index(cuid, aid)
```

如果你以为高枕无忧，那就 too young, too simple 了.　这个方法在 MySQL 5.0 使用是没问题的，但是在之后，5.6之前使用是有bug的，亲自在我的测试库, Percona 版本5.5, 果然是的.
不过还是有办法解决的, 如下

```
set session old_alter_table = on;
alter ignore table register_chn add unique index(cuid, aid);
```

如果你真的想使用上述方法，

```
set session old_alter_table =on;
来解决这个问题，而且有是有主备的情况，记得执行前设置一下.
set session sql_log_bin=off. 
```
以免备库报错，同样，还需要在备库重复一下主库的操作。 

这也算是一个不太完美的解决方法吧.


