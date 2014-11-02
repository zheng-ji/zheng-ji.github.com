---
layout: post
title: "Nginx与PHP-FPM系统参数配置"
date: 2014-11-02 15:46
comments: true
keywords: Nginx 调优 PHP 优化
description: Nginx 调优 PHP 优化
categories: System
---

需要为机器调配参数了, 早些时候看过不少文章, 但没经历过始终不深刻, 以下讲的时在 8core 8G，Centos 配置 PHP-FPM 与 NGINX。

###PHP-FPM 参数配置### 

在Ubuntu系统中 `/etc/php5/fpm/pool.d`目录下, 在centos是`/etc/php-fpm.d`目录下编辑`www.conf`

```
listen = /var/run/php5-fpm.sock
listen.backlog = -1 (on FreeBSD -1 unlimit)
pm = static # 使用静态进程管理
pm.max_children = 128
request_terminate_timeout = 8s #设置太大，会导致work进程过多，来不及kill掉
```
编辑 `/etc/default/php5-fpm`

```
ulimit -n 655360
```

###Nginx 配合参数###

* 启用irqbalance

由于精简系统的服务而没有开启irqbalance服务，但是irqbalance现在被证实为非常有必要的服务，他的主要功能是可以合理的调配使用各个CPU核心，特别是对于目前主流多核心的CPU，简单的说就是能够把压力均匀的分配到各个CPU核心上，对提升性能有很大的帮助，

```
shell> yum -y install irqbalance
shell> service irqbalance start
cat /proc/irqbalance 查看中断的分布
```

* 为nginx 绑定cpu

```
worker_rlimit_nofile 300000;
worker_processes  8;
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
```

* 连接数调整，

nginx发起的连接数，远远超过了 php-fpm 所能处理的数目，导致端口（或socket）频繁被锁，造成堵塞。

```
vi /etc/sysctl.conf 进行了微调，
fs.file-max = 6553600

vim /etc/security/limits.conf
* soft nofile 655360
* hard nofile 655360

vim /etc/nginx/nginx.conf

worker_rlimit_nofile 300000;
events {
    worker_connections 300000;
    use epoll;
}
http {
    keepalive_timeout  0; #关闭keepalive_timeout, 快速释放系统资源
}
```

从查到的资料看起来 8 核 理論值的最大連線數 = `worker_processes * worker_connections / 8`
