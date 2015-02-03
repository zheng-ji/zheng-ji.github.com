---
layout: post
title: "haproxy - MySQl 的负载均衡"
date: 2015-02-03 23:17
keywords: haproxy
description: haproxy mysql 
categories: DataBase
---

服务器上每个PHP进程占用一个数据库链接，当有 n 台服务器, 每台服务器用用100 * m 个PHP 进程的时候，数据库的压力是有点小大。

为了解决这个问题, 可以有的选择是：

+ 业内炒的比较火的有，奇虎[Atlas](https://github.com/Qihoo360/Atlas), 淘宝前架构师写的[OneProxy](http://weibo.com/dbatools), 官方的MySQL-Proxy;
+ 从连接层解决负载均衡的压力，Haproxy 所擅长的事情

对于第一个选择，同事做过调研，使用起来不太放心。官方库就无人维护, 于是，最后选择了 Haproxy 来承担数据库的前端代理.链接数下降明显。

----

### 一些关键的配置

参考[连接](http://www.sysads.co.uk/2014/08/install-haproxy-1-5-6-on-ubuntu-14-04/):

以下是配置内容

```
listen mysql-cluster
    bind 127.0.0.1:3306  # 连接本地3306 到后端的DB
    mode tcp
    option mysql-check user haproxy_check # haproxy_check 是该haproxy用户
    balance roundrobin
    server mysql-1 10.0.0.1:3306 check # 后端DB
    server mysql-2 10.0.0.2:3306 check # 后端DB

listen 0.0.0.0:8080 # 监控页面
    mode http
    stats enable
    stats uri /
    stats realm Strictly\ Private
    stats auth A_Username:YourPassword
    stats auth Another_User:passwd
```

值得注意的是，我们需要在DB 里面添加用户 haproxy_check,使得它有权限访问这个数据库。一开始我习惯用

```
假设我的内网ip是 192.168.1.5
create user 'haproxy_check'@'192.168.1.5' identified by 'xxx';
flush privileges;
```
事实上这样连接 haproxy 会报：

```
mysql -h127.0.0.1 -uusername -p
lost connection to mysql server at 'reading initial communication packet'
```

后来老实按照 digitalocean 的文章修改成

```
INSERT INTO mysql.user (Host,User) values ('192.168.1.5','haproxy_check');
flush privileges;
```

测试就通过了.好奇怪，在我的理解中很不应该, 明天继续看看为什么会这么奇怪。







