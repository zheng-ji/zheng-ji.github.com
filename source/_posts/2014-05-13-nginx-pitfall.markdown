---
layout: post
title: "[译]nginx pitfall"
date: 2014-05-13 22:26
comments: true
categories: Nginx 
keywords: Nginx pitfall
description: Nginx pitfall
---

今天看完[Nginx Pitfall](http://wiki.nginx.org/Pitfalls),热血用了一个下午翻译了的，以下是正文

##Nginx 陷阱##

无论你是nginx的新用户还是老用户都会遇到nginx的一些陷阱，下面我们着重描述这些陷阱，以及如何避免犯错。这些问题多次出现在 #nginx channel on Freenode#上

###[是指南教我这么做的]###

不要轻易相信网上其他的配置指引，除非你知道这样做的真正意义，并且知道怎么清除它们。很多配置其实写的很糟糕，我们根据互联网上的错误配置收集到的陷阱，它们并非来自刻意搜索，而是来自网络上的疑惑问答，这些问题收到的共同回复是，他们因为看了一些错误配置指引所以不同意我们的方法。写此文的目的是为了阐述我们的观点，如果你也遇到如下问题，该文章正好为你准备。


###[Root 放置在Location块内]###

不好的配置

```bash
server {
  server_name www.domain.com;
  location / {
    root /var/www/nginx-default/;
    [...]
  }
  location /foo {
    root /var/www/nginx-default/;
    [...]
  }
  location /bar {
    root /var/www/nginx-default/;
    [...]
  }
}
```

该配置可以正常工作，把 root 放在 location块内可以完美的运行，但是当你开始添加location快，你就发现问题了。如果你给每个loaction 块添加 root，一旦有一个 location 块没有匹配到,那么他就失去了 root ,让我们来看看正确的配置

```
server {
  server_name www.domain.com;
  root /var/www/nginx-default/;
  location / {
    [...]
  }
  location /foo {
    [...]
  }
  location /bar {
    [...]
  }
}
```

###[多条 index 指令]###

不好的配置

```
http {
  index index.php index.htm index.html;
  server {
    server_name www.domain.com;
    location / {
      index index.php index.htm index.html;
      [...]
    }
  }
  server {
    server_name domain.com;
    location / {
      index index.php index.htm index.html;
      [...]
    }
    location /foo {
      index index.php;
      [...]
    }
  }
}
```

为什么要重复多条不需要的 index 指令？事实上只需要用一次，它仅仅需要放置在 `http{}` 块内，后面的配置会继承它。

正确的配置

```
http {
  index index.php index.htm index.html;
  server {
    server_name www.domain.com;
    location / {
      [...]
    }
  }
  server {
    server_name domain.com;
    location / {
      [...]
    }
    location /foo {
      [...]
    }
  }
```

###[使用if]###
这里有小段篇幅是关于使用`if` 表达式，我们需要检查所谓的 “是否有害”，让我们看看那些错误的配置

不好的配置

```
server {
  server_name domain.com *.domain.com;
  if ($host ~* ^www\.(.+)) {
    set $raw_domain $1;
    rewrite ^/(.*)$ $raw_domain/$1 permanent;
    [...]
  }
}
```
这里明显有问题，第一条 `if` 指令就引起我们的注意，为什么是不好的配置呢, 你是否阅读过 [是否有害](http://wiki.nginx.org/IfIsEvil)。

使用if 指令，nginx 会强制检查所有到来的请求，检测每一条指令是否有危害，这是极度低效的。使用两个 server 配置可以避免上述问题。

正确的配置

```
server {
  server_name www.domain.com;
  return 301 $scheme://domain.com$request_uri;
}
server {
  server_name domain.com;
  [...]
}
```

这种方法不仅是配置易读，而且降低了 nginx 的处理负担。我们摆脱了`if`指令的陷阱，我们也使用了 `$scheme` 代替了 URI scheme 是 http 还是 https 的硬编码.

###[判断文件是否存在]###

使用 `if` 指令确保文件是否存在，是糟糕的实践，如果你有机会接触较新的 nginx 版本，查看下`try_files` 指令，该指令更适合做该事情。

不好的配置

```
server {
  root /var/www/domain.com;
  location / {
    if (!-f $request_filename) {
      break;
    }
  }
}
```

正确的配置

```
server {
  root /var/www/domain.com;
  location / {
    try_files $uri $uri/ /index.html;
  }
}
```

我们做的改变是，我们不使用`if`指令，而是使用 `try_files`,如果`$uri`不存在，尝试 `$uri/` 如果不存在，就使用默认的文件 `index.html`

这个场景中，将会测试$uri是否存在，如果存在调用该服务，反之则会测试该目录是否存在，如果不存在就会调用 `index.html`，前提是`index.html`是存在的。这时候仅仅是简单的加载该页面。

###[包的前端控制器模式]###

“前端控制器模式”的设计很流行并广泛应用于 PHP 软件包，不乏很多比它更复杂的例子，使用 Drupal, Joomla 等，你只需要使用

```
try_files $uri $uri/ /index.php?q=$uri&$args;
```

注意： - 参数名会依据你所使用的包不同而做相应的改变

* `q`用于Drupal, Joomla, WordPress
* `page` 用于CMS Made Simple

一些软件甚至不需要查询字符串，可以通过`REQUEST_URI`获取（比如WordPress就支持）

```
try_files $uri $uri/ /index.php;
```

当然，你的情况可能有所不一样，你可能需要更复杂的配置。对于简单的站点，该配置已经完美的支持，通常我们都是由浅入深地学习一样东西 。

如果你不关心目录是否存在，你可以决定跳过该目录检查,并且移 `$uri/`

###[传递不受控制的请求给PHP]###
很多 PHP web 站点的 Nginx 配置要求，每一个URI需要附带`.php`给 PHP 解释器，注意到这里有一个关于PHP设置的严重安全隐患，因为它允许第三方执行任意代码。

比如

```
location ~* \.php$ {
  fastcgi_pass backend;
  ...
}
```

这里每一个关于.php的请求将会传给 `FastCGI` 后端，这是PHP默认配置，在不知道文件的具体路径下,试图通过该配置执行该文件。

举个例子，如果请求 `/forum/avatar/1232.jpg/file.php`不存在， PHP 解释器将会返回 `forum/avatar/1232.jpg`，如果该文件有内嵌php代码，就会相应地被执行了。

避免上述情况的配置选项是：

* 设置 php.ini 里的 `cgi.fix_pathinfo=0`，该配置使得 php 解释器仅仅执行具体制定的文件，停止执行不存在的文件。
* 确保 nginx 指定具体的php可执行文件

```
location ~* (file_a|file_b|file_c)\.php$ {
  fastcgi_pass backend;
  ...
}
```
* 禁止执行用户上传目录中的php文件 

```
location /uploaddir {
  location ~ \.php$ {return 403;}
  ...
}
```

* 使用 `try_files`指令，过滤掉有问题的条件

```
location ~* \.php$ {
  try_files $uri =404;
  fastcgi_pass backend;
  ...
}
```

* 使用嵌套块过滤有问题的条件

```
location ~* \.php$ {
  location ~ \..*/.*\.php$ {return 404;}
  fastcgi_pass backend;
  ...
}

```

###[脚本文件名中的 FastCGI 路径]###
外界很多配置指引依靠绝对路径来获取你的信息，在PHP配置块内经常存在，当你从软件仓库中安装 nginx,他的配置文件中会有 "include fastcgi_params",该文件在 nginx 文件夹 /etc/nginx 的目录下。

正确的配置

```
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
```

不好的配置

```
fastcgi_param  SCRIPT_FILENAME    /var/www/yoursite.com/$fastcgi_script_name;
```

`$document_root` 在哪里呢？他在`server`块中`root`指令，如果你的`root` 指令配置不存在， 请回头看看 `第一个陷阱`。


###[麻烦的重写]###
不要感到沮丧，你很容易被正则表达式迷糊住。事实上，我们可以努力让配置干净简洁不添加毫不相干的东西。

不好的配置

```
rewrite ^/(.*)$ http://domain.com/$1 permanent;
```

不好的配置

```
rewrite ^ http://domain.com$request_uri? permanent;
```

正确的配置

```
return 301 http://domain.com$request_uri;
```

反复看这几个例子，OK，第一个重写捕获斜线前完整的URI。通过使用内置的变量 `$request_uri` 我们可以有效地避免做任何捕获或匹配,并通过返回指令,我们可以完全避免对正则表达式的使用。


###[重写时丢失`http://`]###
很简单，重写都是相互关联的，除非你特意设置nginx。重写规则挺简单的，仅仅添加一条规则。

不好的配置

```
 rewrite ^/blog(/.*)$ blog.domain.com$1 permanent;
```

正确的配置

```
rewrite ^/blog(/.*)$ http://blog.domain.com$1 permanent;
```

上述看到的例子，我们仅仅是给该条规则添加了`http://`，便实现了重写，很简单，也很实用。

###[代理一切]###

不好的配置

```
server {
    server_name example.org;
    root /var/www/site;
 
    location / {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/tmp/phpcgi.socket;
    }
}
```

这个例子中，你转发请求给php,如果是 apache 服务器会这么干。但是 nginx 服务器不需要这样做，`try_files`指令会按顺序检测文件。这意味着nginx可以首先寻找需要测试的静态文件，如果找不到才返回用户指定的文件。通过这样的方式，php解释器不会执行任意php文件，除非你拥有该请求路径的php文件，而且能帮你节约资源，特别是当你直接通过php请求一个大小为1MB的图片1千次。

正确的配置

```
server {
    server_name example.org;
    root /var/www/site;
 
    location / {
        try_files $uri $uri/ @proxy;
    }
 
    location @proxy {
        include fastcgi.conf;
        fastcgi_pass unix:/tmp/php-fpm.sock;
    }
}
```

或者

```
server {
    server_name example.org;
    root /var/www/site;
 
    location / {
        try_files $uri $uri/ /index.php;
    }
 
    location ~ \.php$ {
        include fastcgi.conf;
        fastcgi_pass unix:/tmp/php-fpm.sock;
    }
}
```


如果被请求的 URI 存在就可以被 nginx 返回，如果不存在，那么是否存在一个具有index文件的目录里，同时，我们是否已经为该请求配置上了 index 指令。如果仍然不存在，重写规则将发送`index.php`到你的后端，只有当nginx前端不能处理你的请求，才会让后端服务参与进来。

想想有你的请求中有多少是静态资源，比如图片，css，javascript,我们可以通过上述方式的配置来节约这些资源。


###[配置改变，并没有生效]###

你的配置很完美，但是你依然捶胸顿足。问题出现在你的浏览器缓存上，当你加载一些东西，浏览器会保存下来，也会记住了它是如何请求服务的，如果你使用 `types{}`块，你会遇到如下问题。

修复

[选项1] 在火狐中按下 `Ctrl+Shift+Delet` ,检测缓存，并且清空。其他的浏览器自行搜索清空缓存的方法，每一次修改配置，记得清空缓存，除非你确认不需要这么做，这一步骤会帮你避免很多头痛。

在火狐中，你也可以选择一个更长久的解决方法，在URI搜索条内，输入`about:config`,然后搜索 `browser.cache.check_doc_frequency` ，设置其值为1，这样没加载一次包就会检测。

[选项2] 使用 `curl`
如果还行不通，而且你是在`vritualbox` 虚拟机里面运行 nginx,那么，可能是 `sendfile()` 惹得祸，你只需要注释掉 sendfile 指令，或者将其设置为 `off`,这个指令在`nginx.conf`里。

```
sendfile off
```

###[HTTP头部丢失]###

如果你没有显性设置 

```
 underscores_in_headers on
```

nginx 将会默认用下划线去掉http头部（这符合http标准），这么做是为了防止在映射CGI头信息时候有歧义。这个过程中破折号和下划线都被映射成了下划线。
