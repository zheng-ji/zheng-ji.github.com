---
layout: post
title: "Sysdig 值得拥有"
date: 2015-09-10 23:10
comments: true
categories: System
description: sysdig
---

定位服务器问题时,  我们需要各式各样的武器, 诸如 iftop, ifstat, netstat, tcpdump, iostat。dstat 等, 因此工具箱需要装满很多工具, 在面对问题的时候才能显得不费吹灰之力, 迅速定位问题并解决, 保障服务稳定运行。Sysdig 的横空出世, 对我们而言, 就是一把瑞士军刀, 灵活小巧, 武艺多端.

* [安装](#第一节)
* [常用操作](#第二节)
* [高效的实战](#第三节)

<h3 id="第一节">安装</h3>

```
curl -s https://s3.amazonaws.com/download.draios.com/DRAIOS-GPG-KEY.public | apt-key add -
curl -s -o /etc/apt/sources.list.d/draios.list http://download.draios.com/stable/deb/draios.list
apt-get update
apt-get -y install sysdig
```

<h3 id="第二节">常用操作</h3>

sysdig 有很多 chisel, chisel 意为 铲子, 可以理解为定位某类问题的工具, sysdig 采用 Lua 编写的。

* 查看 chisel 列表

```
sysdig -cl  
```

* 查看具体某个 chisel 的提示

```
sysdig -i spy_logs
```

* 使用某个 chisel

```
sysdig -c spy_logs
```

* 过滤器可以帮助我们从各种输出信息中, 筛选出我们需要的, 比如 
`proc.name=foo` , 
如果你记住不了太多过滤器也无妨,  我们可以借助如下命令查看过滤器

```
sysdig -l
```

* 记录定位信息到文本, 以及从文本读取信息

```
sysdig -w tracefile.cap
sysdig -r tracefile.dump proc.name=sshd
```

<h3 id="第三节">高效的实战</h3>

* 服务器上经常需要查看哪个服务带宽占用使用较高, 特别是被 DDOS 的时候。

```
sudo sysdig -c topprocs_net
```

* 查看某个IP的通讯数据,并以ASCII 码输出

```
sudo sysdig -s2000 -A -c echo_fds fd.cip=127.0.0.1
```

* 查看非请求 redis-server 的其他请求进程和句柄

```
sudo sysdig -p"%proc.name %fd.name" "evt.type=accept and proc.name!=redis-server"
```

* 查看访问该服务器的所有 GET 请求数据

```
sudo sysdig -s 2000 -A -c echo_fds fd.port=80 and evt.buffer contains GET
```

* 查看访问该服务器的 SQL 语句

```
sudo sysdig -s 2000 -A -c echo_fds evt.buffer contains SELECT
```

* 查看磁盘读写最高的进程

```
sysdig -c topprocs_file
```

* 查看延迟最大的系统调用

```
sysdig -c topscalls_time
```

* 查看具体文件的操作细节

```
sysdig -A -c echo_fds "fd.filename=syslog"
```

* 查看 IO 延迟大于 1ms 的文件

```
sudo sysdig -c fileslower 1
```

* 监视某个文件是否被操作, 从安全出发想象空间很大哦

```
sudo sysdig evt.type=open and fd.name contains /etc
```

---

[Sysdig 官网](http://www.sysdig.org/wiki/)
