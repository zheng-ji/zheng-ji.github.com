---
layout: post
title: "my.cnf 配置依据"
date: 2015-07-12 08:35
comments: true
categories: DataBase
description: MySQL my.cnf
---

阅读完[构建高性能服务器](http://book.douban.com/subject/6873681/) 一书，书中对 `MySQL` 配置做了讲解，我用烂笔头记录一番，明白它们这么配置的真正意义，形成系统的认知。

* [通用配置](#第一节)
* [innodb_pool_buffer 合理配置](#第二节)
* [文件打开数的合理设置](#第三节)
* [打开表优化设置](#第四节)
* [临时表的设置](#第五节)
* [查看索引命中率](#第六节)


<h3 id="第一节">通用配置</h3>

```
skip-name-resolve:
```

禁止 MySQL 对外连接 DNS 解析 ,这一选项可以消除 MySQL 进行DNS解析时间，开启该选项，所有远程主机连接授权都要使用 IP。

```
back_log=512
```

指出在 MySQL 暂停响应之前，有多少个请求被存在堆栈中。

```
key_buffer_size
```

指定用于索引的缓冲区大小，增加它可得到更好的索引处理能力，对于内存 4G 左右，可设置为 256MB。

```
max_allowed_packet=4M
```
设定在一次网络传输中的最大值。

```
thread_stack=256k
```
设置每个线程的堆栈大小，可满足普通操作。

```
table_cache=614k
```
高速缓冲区的大小，当mysql访问一个表示，缓冲区还有空间，那么这个表就会被放入缓冲区，一般看峰值时间状态值，open_tables与open_cache，用于判断是否需要增加table_cache，如果open_Table接近table_cache就要增加了。

```
sort_buffer_size=6M
```
设定查询排序所用的缓冲区大小，是对每个链接而言，如果有100个连接，那么分配的总缓存是100*6=600M。
read_buffer_size,join_buffer_size 都是对单个链接而言。

```
thread_cache_size=64
```
设置缓存中的链接线程的最大数量,4GB内存以上的我们会给64或者更大。

```
query_cache_size=64M
```

如果该值比较小反而会影响效率，可以考虑不用。如果太大，缓存区中碎片会很多。

```
tmp_table_size
```

设置内存临时表的最大值，如果超过改制，就会将临时表写入磁盘，范围是1kB-4GB。

```
max_connection
```

如果超过该值，会出现`too many connections`。

```
max_connect_errors
```
设置每个主机中连接请求异常中断最大次数，如果超过，MySQL禁止host连接请求，直到MySQL重启。

```
wait_timeout = 120
```
一个请求的最大链接时间。

```
thread_concurrency=8
```
该参数为服务器逻辑CPU * 2。

```
skip-networking
```
该选项可以关闭连接方式，如果需要远程连接，不要开启。

```
innodb_flush_log_at_trx_commit = 1
```
设置为0就是等到 innodb_log_buffer_size 队列满了之后再统一存储，默认是1，是最安全的设置。

```
innodb_thread_concurrency = 8
```
服务器几个CPU就设置多少。

```
read_rnd_buffer_Size = 16M
```

设置随机读的使用的缓冲区。

<h3 id="第二节">innodb_buffer_pool 的合理设置</h3>

不要武断的把innodb_buffer pool 配置为内存的50-80% 应具体而定。

```
show status like 'innodb_buffer_pool_%'

关注的有
innodb_buffer_pool_pages_data;
innodb_buffer_pool_page_total;
innodb_buffer_pool_read_request;
innodb_buffer_pool_reads;
```

然后看

```
读命中率 (innodb_buffer_pool_read_request - innodb_buffer_pool_reads) / innodb_buffer_pool_read_request;
写命中率 innodb_buffer_pool_pages_data /innodb_buffer_pool_page_total;
```
如果读写命中率都能在95%以上就很好了。


<h3 id="第三节">文件打开数的合理设置依据</h3>

```
show global status like 'open_files'
show variables like 'open_file_limit';
```
比较合适的设置是 Open_files / open_files_limit * 100% <= 75;


<h3 id="第四节">打开表优化设置依据</h3>

```
show global status like 'open%tables';
show variable like 'table_cache';
```
比较合适

```
open_tables / opende_tables * 100 > 85%;
open_Tables / table_Cache* 100% < 95%；
```

<h3 id="第五节">临时表的设置依据</h3>

每次执行语句，关于已经被创造了的临时表的数量，可以这么设置:

```
show gloabl status like 'created_tmp5';
Created_tmp_disk_table / Created_tmp_tables * 100% <= 25 %
```


<h3 id="第六节">查看索引命中率</h3>

```
show global status like 'key_read%';
```
key_cache_miss_rate = key_Read / key_read_request * 100% 小于 1% 是很好的，
意味着1000个请求有一个直接读硬盘。
