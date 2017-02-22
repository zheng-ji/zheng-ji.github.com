---
layout: post
title: "Squid 做正向代理"
date: 2017-02-22 22:30
comments: true
categories: System
description: squid, 代理
---

### 正向代理和反向代理

* 正向代理

正向代理 是一个位于客户端和原始服务器之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定原始服务器，然后代理向原始服务器转交请求并将获得的内容返回给客户端。客户端必须要进行一些特别的设置才能使用正向代理。

* 反向代理

反向代理正好相反，对于客户端而言它就像是原始服务器，并且客户端不需要进行任何特别的设置。客户端向反向代理内容发送普通请求，接着反向代理将判断向何处(原始服务器)转交请求，并将获得的内容返回给客户端，就像这些内容原本就是它自己的一样。

### 正向代理软件

Nginx 能做正向代理，遗憾的是它不能做 HTTPS 的正向代理。以下是一个例子。

* 不能有 hostname。 
* 必须有 resolver, 即 DNS
* 配置正向代理参数，均是由 Nginx 变量组成

```
server{
      resolver 8.8.8.8;
      resolver_timeout 30s; 
      listen 80;
      location / {
                proxy_pass http://$http_host$request_uri;
                proxy_set_header Host $http_host;
        }
}
```

Squid 可正向代理 HTTP 以及 HTTPS


```
sudo apt-get install squid
```

编辑 `/etc/squid3/squid.conf`, 并重启

```
http_port 3128                 	#代理服务器的端口
#http_access deny !Safe_ports 	#注释掉此项
#http_access deny manager     	#注释掉此项

#添加下面两项，设置哪些网段可以访问本代理服务器
acl our_networks src 172.16.1.0/24 
http_access allow our_networks
```

```
sudo service squid3 restart
```

Squid 的层次代理值得一提， 若我们需要定期地切换代理服务器的话, 启动一个 Squid 代理, 而这个代理会将请求转发到其他代理上面. 然后我们只需定时更新本地 Squid 代理的配置文件, 然后重启这个本地代理即可. 层次代理用到了 `cache_peer` 这个配置文件

```
cache_peer hostname type http_port icp_port option

e.g.
cache_peer xxx.proxy.com parent 9020 0 no-query default login=xxxxx:yyyy
```

* hostname: 指被请求的同级子代理服务器或父代理服务器。可以用主机名或ip地址表示；
* type：指明 hostname 的类型，是同级子代理服务器还是父代理服务器，也即 parent 还是 sibling；
* http_port：hostname的监听端口；
* icp_port：hostname 上的ICP监听端口，对于不支持ICP协议的可指定7；
* options：可以包含一个或多个关键字。
   1. proxy-only：指明从peer得到的数据在本地不进行缓存，缺省地，squid是要缓存这部分数据的；
   2. weight=n：用于有多个peer的情况，如果多于一个以上的peer拥有你请求的数据时，squid通过计算每个peer ICP 响应时间来 决定其weight的值，然后 squid 向其中拥有最大 weight 的peer发出ICP请求。
   3. no-query：不向该peer发送ICP请求。如果该peer不可用时，可以使用该选项；
   4. Default：有点象路由表中的缺省路由，该peer将被用作最后的尝试手段。当你只有一个父代理服务器并且其不支持ICP协议时，可以使用default和no-query选项让所有请求都发送到该父代理服务器；
* login=user:password：当你的父代理服务器要求用户认证时可以使用该选项来进行认证
