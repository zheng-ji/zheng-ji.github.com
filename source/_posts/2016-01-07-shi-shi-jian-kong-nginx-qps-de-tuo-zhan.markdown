---
layout: post
title: " 实时监控 nginx qps 的拓展"
date: 2016-01-07 12:59
comments: true
categories: System
description: Nginx lua
---

用下班时间写了一个 ngx lua 的拓展, [GitHub](https://github.com/zheng-ji/ngx_lua_reqstatus)。可以:

- [x] 实时监控域名的 qps
- [x] 实时监控域名的 5xx 个数
- [x] 实时监控域名的 响应时长
- [x] 并附带 Ganglia 监控插件

## 使用

*  配置 `nginx.conf`

```
http {
    ...
    ...

    lua_shared_dict statics_dict    1M; # 初始化变量
    lua_package_path "/etc/nginx/ngx_lua_reqstatus/?.lua";  #路径
    log_by_lua_file "/etc/nginx/ngx_lua_reqstatus/hook.lua"; # 添加此句

    server {
        listen 80;
        server_name  justforfun.com; 

        location /{
            ...
        }
    }
    # 监控服务
    server {
        listen 127.0.0.1:6080;
        location /{
            access_by_lua_file "/etc/nginx/ngx_lua_reqstatus/status.lua";
        }
    }
}
```

* 效果

```
curl localhost:6080/?domain=justforfun.com
```

* 输出

```
Server Name:    justforfun.com
Seconds SinceLast:   1.4399998188019 secs
Request Count:      1
Average Req Time:   0 secs
Requests Per Secs:  0.69444453182781
5xx num:    0
```


