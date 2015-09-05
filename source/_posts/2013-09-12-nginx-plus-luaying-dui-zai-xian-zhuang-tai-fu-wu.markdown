---
layout: post
title: "nginx+lua应对在线状态服务"
date: 2013-09-12 23:12
comments: true
categories: Server
description: nginx lua
---

几天前在@ideawu的博客上看到了其一篇文章中讲到的技术题.
[《在线状态服务在网站系统中的应用》](http://www.ideawu.net/blog/archives/732.html) 。于是内心折腾的火焰便熊熊燃烧起来，连续几天晚上折腾到凌晨1点。感谢@agentzh 春哥与阿里巴巴的技术贡献,让我可以安心睡觉 :) 

在线状态服务这样的场景在互联网应用上比比皆是

* 聊天系统中同时在线的人数
* 游戏中同时在线玩家等

文中所言,现实的在线状态应用场景是:
网页中用JavaScript启动一个定时器, 定期报告在线状态, 也就是向在线状态服务器发送心跳包.对某个同时在线100万人, 每天1亿PV的网站来说, 在线状态服务器一天接收到的心跳包大概是10亿个, 也即每秒10000个请求(10000qps).要实现这样的在线状态服务器, 也是一个挑战.
 
好了，我当时真的很兴奋，感觉自己像接到了任务，立马打起了鸡血研究. 脑海中涌现出来的直觉是，使用tornado + Redis，这样的高并发组合应对这样的场景应该是无压力的，但是因为这样的事情我做过，想挑战自己非舒适的区域，挖掘新的知识大陆。
我们一直都说，没有经过打磨的想法一般都是廉价的。后来我想到的是Ngnix这样的服务器来抗如此高的请求，那么我们就自己写个拓展，查询内存数据库，这里我选择用redis来做kv内存数据库，

文中谈到的自己设计精巧的kv数据库，甚至用C++写针对性的http服务，来提升通用服务器的性能。曾经写过一个KV数据库，其性能真的是渣渣。[源码链接](https://github.com/zheng-ji/KvDb)。欢迎吐槽.
一直以来听闻春哥的nginx lua模块拓展，可以为nginx定制业务。想想以前用c语言实现，简直就是痛苦,这一次可以使用lua，甚为欢悦，没有什么比探索新东西带来的乐趣更令人振奋了

首先要有lua的运行环境。他是一种胶水粘合剂 ：） 还必须安装**luajit**

```
#设置环境变量，后续安装需要
shell> export LUAJIT_LIB=/usr/local/lib
shell> export LUAJIT_INC=/usr/local/include/luajit-<VERSION>
```

#### 需要安装的nginx 插件

```
ngx_devel_kit https://github.com/simpl/ngx_devel_kit
set-misc-nginx-module https://github.com/agentzh/set-misc-nginx-module
memc-nginx-module https://github.com/agentzh/memc-nginx-module 
lua-nginx-module https://github.com/chaoslawful/lua-nginx-module
```

#### 下载之后，安装nginx插件
                 
```
./configure --prefix=/path/to/nginx_src --add-module=/path/to/ngx_devel_kit --add-module=set-misc-nginx-module \
--add-module=/path/to/lua-nginx-module -add-module=ngx_devel_kit 
make 
make install
```

nginx.conf 文件                 

```
location /echo {  
    default_type 'text/plain';  
    echo 'hello echo';  
}  

location /lua {  
    default_type 'text/plain';  
    content_by_lua 'ngx.say("hello, lua")';  
}  
```

也许你会遇到一些麻烦,执行如下命令

```
shell> echo "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
shell> ldconfig
```

重启服务器之后，应该就可以看到hello,lua :) (到这里仅仅是开始，折腾了有点久)
 
### 好了，来完成ideawu的作业 ：）

>在线状态服务, 是这样的一个服务, 它维护了网站当前的在线用户列表, 接受其它模块的查询. 是实现统计网站同时在线人数, 维护在线用户列表等功能的基础服务. 在Facebook的聊天系统中, 在线状态是为聊天系统服务的, 所以在线状态是一种”强”在线, 也即用户保持着和Comet服务器的连接, 可随时接受服务器推送(push)的消息.在高并发请求的情况下如何完成该需求呢

使用 Lua 脚本语言操作 Redis。这里使用 content_by_lua_file （nginx_lua_module 模块具有）来引入 Lua 脚本文件。
agentzh 提供了一个很方便的开发包，如下：[lua-resty-redis](https://github.com/agentzh/lua-resty-redis.git)

该包中，有一个 lib 目录，将 lib 目录下的文件和子目录拷贝至目录 /home/zj/soft/data/www/lua/
在 Nginx 配置文件中，需要加一行代码，以便引入 redis.lua。
注：加在 http 段里。

```
lua_package_path "/home/zj/soft/data/www/lua//?.lua;;";  
```

为了使得 lua 脚本的修改能及时生效，需要加入一行代码，如下：注：在 server 段里，加入代码，如果不加此代码或者设置为 on 时，则需要重启 Nginx。 不过nginx 会报警

```
lua_code_cache off;  
```

nginx.conf 里

```
location /getolNum {
    content_by_lua_file /conf/online.lua;  
} 
```

##### online.lua 源码

```lua
local redis = require "redis"
local cache = redis.new()

local ok, err = cache.connect(cache, '127.0.0.1', '6379')
if not ok then
    ngx.say("failed to connect:", err)
    return
end

cache:set_timeout(30000)
args = ngx.req.get_uri_args()
user = args["user"]
--设置用户 3 min 过期
cache:setex(user,180,23)

local res,err = cache:keys('*')

--count res num
count = 0
for _ in pairs(res) do
    count = count + 1
end
ngx.say(count)
local ok, err = cache:close()
```

结果

```
curl localhost/getOlnum?user=zj
返回：1
curl localhost/getOlnum?user=zhengji
返回：2
```

Happy Hacking :)
 








