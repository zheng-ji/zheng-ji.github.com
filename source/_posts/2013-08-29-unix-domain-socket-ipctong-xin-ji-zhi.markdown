---
layout: post
title: "Unix Domain Socket – IPC通信机制"
date: 2013-06-17 23:52
comments: true
categories: NetWork
---
#### 什么是Unix Domain Socket
基于socket的框架上发展出一种IPC机制，就是UNIX Domain Socket。虽然网络socket也可用于同一台主机的进程间通讯（通过loopback地址127.0.0.1），但是UNIX Domain Socket用于IPC更有效率：

* 不需要经过网络协议栈
* 不需要打包拆包、计算校验和、维护序号和应答等

只是将应用层数据从一个进程拷贝到另一个进程。这是因为，IPC机制本质上是可靠的通讯，而网络协议是为不可靠的通讯设计的。UNIX Domain Socket也提供面向流和面向数据包两种API接口，类似于TCP和UDP，但是面向消息的UNIX Domain Socket也是可靠的，消息既不会丢失也不会顺序错乱。

#### 应用

UNIX Domain Socket是全双工的，API接口语义丰富，相比其它IPC机制有明显的优越性，目前已成为使用最广泛的IPC机制，比如X Window服务器和GUI程序之间就是通过UNIX Domain Socket通讯的。

Nginx通过unix:/socket与fastcgi连接，提升性能,比tcp socket要高效
+ 在nginx.conf中修改配置为：

```
fastcgi_pass unix:/tmp/php-cgi.sock;
#fastcgi_pass 127.0.0.1:9000;
```

+ 在php-fpm.conf中修改配置为：

```
/tmp/php-cgi.sock
```

通信过程

{% img /images/2013/08/ipc_unix_socket-300x300.png %}

用Unix domain socket写的Demo

使用UNIX Domain Socket的过程和网络socket十分相似，也要先调用socket()创建一个socket文件描述符，address family指定为AF_UNIX，type可以选择SOCK_DGRAM或SOCK_STREAM，protocol参数仍然指定为0即可。通常是指定/tmp/目录下的一个文件作为通信路径

源码demo[链接](https://github.com/zheng-ji/ToyCollection/tree/master/unix-sock)

