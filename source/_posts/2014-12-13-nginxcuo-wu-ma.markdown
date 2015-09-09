---
layout: post
title: "Nginx 错误码"
date: 2014-12-13 15:18
comments: true
categories: System
description: Nginx 错误码
---

在定位线上服务问题的时候，通常会去查看`Nginx` 的`error log`

那么 error 的定义, 对查找问题就显得很有帮助

* upstream prematurely closed connection

>请求uri的时候出现的异常，是由于 upstream 还未返回应答给用户时用户断掉连接造成的，对系统没有影响，可以忽略

* recv() failed (104: Connection reset by peer) 

>服务器的并发连接数超过了其承载量，服务器会将其中一些连接Down掉；客户关掉了浏览器，而服务器还在给客户端发送数据;

* (111: Connection refused) while connecting to upstream 

>用户在连接时，若遇到后端 upstream 挂掉或者不通，会收到该错误

* (111: Connection refused) while reading response header from upstream 

>用户在连接成功后读取数据时，若遇到后端 upstream 挂掉或者不通，会收到该错误

* (111: Connection refused) while sending request to upstream 

>Nginx 和 upstream 连接成功后发送数据时，若遇到后端 upstream 挂掉或者不通，会收到该错误

* (110: Connection timed out) while connecting to upstream 

>nginx 连接后面的 upstream 时超时

* (110: Connection timed out) while reading upstream 

>nginx 读取来自 upstream 的响应时超时 

* (110: Connection timed out) while reading response header from upstream 

>nginx 读取来自 upstream 的响应头时超时

* (110: Connection timed out) while reading upstream 

>nginx读取来自 upstream 的响应时超时

* (104: Connection reset by peer) while connecting to upstream 

>upstream发送了 RST，将连接重置

* upstream sent invalid header while reading response header from upstream 

>upstream 发送的响应头无效

* upstream sent no valid HTTP/1.0 header while reading response header from upstream

>upstream 发送的响应头无效

* client intended to send too large body 

>用于设置允许接受的客户端请求内容的最大值，默认值是1M，client 发送的 body 超过了设置值

* reopening logs 

>用户发送kill  -USR1命令

* gracefully shutting down

>用户发送kill  -WINCH命令

* no live upstreams while connecting to upstream 

>upstream 下的 server 全都挂了


* SSL_do_handshake() failed

>SSL握手失败

* ngx_slab_alloc() failed: no memory in SSL session shared cache

>ssl_session_cache大小不够等原因造成

* could not add new SSL session to the session cache while SSL handshaking

>ssl_session_cache 大小不够等原因造成
