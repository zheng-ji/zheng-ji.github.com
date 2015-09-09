---
layout: post
title: "最近用到的 MySql 语句"
date: 2013-12-05 23:01
comments: true
categories: DataBase
keywords: 优化
description: mysql优化
---
该文章没有什么创造性营养，也没有什么技术沉淀，各位看官轻轻带过，仅仅经验之谈，看过用过的人都会，简单的事情就平滑地带过。怎么让索然无味的文章更好地被他人深入阅读,要么就是实用，要么就是深刻，不扯淡, 直奔主题。

### 查看表结构

```
desc Tbl;
show create table Tbl;
show full fields table Tbl;
```

如果表有很多分区，导致很多打点刷屏，那么可以用

```
desc Tbl/G;
```

表太多的话，我的场景是2000张表,:(

```
SHOW TABLES LIKE '%tb%';
```

####查询数据库的字符集

```
show variables like '%character%';
```

####指定字符集直连

```
mysql -h10.145.135.234 -uoss -pxxx dbName --default-character-set=gbk;
```

####赋予权限

```
show grants for 'xxx'@'xxx.xxx.xxx.xxx'
```

####如果想备份，或者复制表

```
create table newtb select * from oldtable
```

####修改表字段

```
alter tbA modify fieldA int(11) unsigned default '0'
```

####移除字段

```
alter table t2 drop column c;
```

####修改主键

```
alter table tbA drop primary key;
Alter table tbA add primary key(dtStatDate,File2)
```

####查询当前数据库的查询进程,干掉挂起的query

```
show processlist;
kill pid
```

####直接在shell外部命令行执行:

```
mysql -h10.112.111.111 -uxxx -pxxxx dbname -Ns -e"
select distinct(iUin) from TableA where dtEventTime >= '2013-05-05' 
" > records.txt;
```

#### 排除重复记录

```
INSERT IGNORE into
Replace Into 是为了让主键替换原有的记录
```

####load data infile语句
从一个文本文件中以很高的速度读入一个表中。使用这个命令之前，mysqld进程（服务）必须已经在运行。为了安全原因，当读取位于服务器上的文本文件时，文件必须处于数据库目录或可被所有人读取。另外，为了对服务器上文件使用load data infile，在服务器主机上你必须有file的权限。

```
load data infile "/home/Order txt" into table Orders(Order_Number, Order_Date, Customer_ID) terminated by',';
```

#### mysqldump 中解决 报"Access denied for user when using LOCK TABLES"

```
mysqldump -udbuser -p dbname --skip-lock-tables > dbname.sql
```

####监控数据库磁盘增长量

```
use information_schema;
#库的大小
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data  from TABLES where table_schema='db1';
```

####时间函数

```
FROM_UNIXTIME(iLoginTime,'%Y-%m-%d')
unix_timestamp('2013-12-03');
Date_SUB(OrderDate,INTERVAL 2 DAY)
Date_ADD(OrderDate,INTERVAL 2 DAY)
timestampdiff(week,’2009-01-24′,’2009-06-20′);
```

####no exist,having
适合做留存用户分析

```
SELECT distinct field1
FROM table a
where date <= 20130430 and
not exists (
        SELECT field1 FROM table b where date>=20130501 and a.field1=b.field1
)

#举例子,可以计算第一次创建用户的人
select * from TbRole having min(CreateTime) between 'xxx' and 'xxx' 
```

####mysql 逻辑运算,好像在编程的样子

```
case condition1
when result1 then 'xx'
when result2 then 'xx'
else result1 then 'xx'
end as 'xx'

ifnull(a,b) #如果a不为空的情况下执行a,否则执行b
if(condition,a,b)#如果符合condition 执行a,否则执行b
```

####妙用group by 
可以在group by 里面做逻辑运算, 缩小逻辑范围

```
select * From tableA group by (round(field1/100,0));
```

####格式化字符串类型

```
concat('a','b')
```

####mysql的临时变量'@'
以下sql是取各道具排前10的等级

```
select * from (
    select dtStatDate,iRoleLevel,iUserNum,
    if(@templevel=a.iRoleLevel,@tno:=@tno+1,@tno:=1) as tno,@templevel:=a.iRoleLevel 
    from tbl_a , (SELECT @tno:= 0,@templevel:=null) tbl_b
    order by a.iRoleLevel asc,a.iUserNum desc
) c
where tno<=10 order by iRoleLevel asc
```

####其他
union all 效率会比union高,因为union有去重功能,会耗时 
