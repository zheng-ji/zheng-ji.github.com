---
layout: post
title: "Nginx 模块开发"
date: 2013-02-17 14:04
comments: true
categories: Server
description: nginx 模块开发
---
写这篇文章的时候，参考[链接](http://blog.codinglabs.org/articles/intro-of-nginx-module-development.html)，特此鸣谢

#### 为何需要Ngnix 模块：

>对于一些访问量特别大，业务逻辑也相对简单的Web调用来说，通过一个nginx module来实现是一种比较好的优化方法。实现一个nginx module实际上比较简单。（@TimYang）

#### Nginx 配置结构了解
Nginx的配置文件是以block的形式组织的，一个block通常使用大括号“{}”表示。block分为几个层级，整个配置文件为main层级，这 是最大的层级；在main层级下可以有event、http等层级，而http中又会有server block，server block中可以包含location block

#### Nginx模块工作原理概述
Nginx本身做的工作实际很少，当它接到一个HTTP请求时，它仅仅是通过查找配置文件将此次请求映射到一个location block，而此location中所配置的各个指令则会启动不同的模块去完成工作，因此模块可以看做Nginx真正的劳动工作者。通常一个 location中的指令会涉及一个handler模块和多个filter模块（当然，多个location可以复用同一个模块）。handler模块负 责处理请求，完成响应内容的生成，而filter模块对响应内容进行处理。因此Nginx模块开发分为handler开发和filter开发

####展示一个简单的Nginx模块开发全过程
我们开发一个叫echo的handler模块，这个模块功能非常简单，它接收“echo”指令，指令可指定一个字符串参数，模块会输出这个字符串作为HTTP响应。例如，做如下配置：

```
location /echo {
    echo "hello nginx";
}
```

首先我们需要一个结构用于存储从配置文件中读进来的相关指令参数，即模块配置信息结构。根据Nginx模块开发规则这个结构的命名规则为ngx_http_[module-name]_[main|srv|loc]_conf_t

实现这个功能需要三步：
+ 读入配置文件中echo指令及其参数；
+ 进行HTTP包装（添加HTTP头等工作）；
+ 将结果返回给客户端。

#### Nginx模块的安装
Nginx不支持动态链接模块，所以安装模块需要将模块代码与Nginx源代码进行重新编译。安装模块的步骤如下：

+ 编写模块config文件，这个文件需要放在和模块源代码文件放在同一目录下。文件内容如下：

```
ngx_addon_name=模块完整名称
HTTP_MODULES=”$HTTP_MODULES 模块完整名称”
NGX_ADDON_SRCS=”$NGX_ADDON_SRCS $ngx_addon_dir/源代码文件名”
```

+ 进入Nginx源代码，使用下面命令编译安装

```
./configure --prefix=安装目录 --add-module=模块源代码文件目录
make
make install
```

这样就完成安装了，例如，我的源代码文件放在/home/zj/tmp/ngx/ngx_http_echo下，我的config文件为：

```
ngx_addon_name=ngx_http_echo_module
HTTP_MODULES="$HTTP_MODULES ngx_http_echo_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS $ngx_addon_dir/ngx_http_echo_module.c"
```

#### 编译安装命令为：

```
./configure --prefix=/usr/local/nginx --add-module=/home/zj/tmp/ngx/ngx_http_echo
make
sudo make install
```

这样echo模块就被安装在我的Nginx上了，下面测试一下，修改配置文件，增加以下一项配置：

```
location /echo {
    echo "This is my first nginx module!!!";
}
```

然后用curl测试一下：

```
curl -i http://localhost/echo
```
