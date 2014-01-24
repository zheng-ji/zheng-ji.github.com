---
layout: post
title: "FastCGI + Nginx + C"
date: 2012-07-05 23:22
comments: true
categories: NetWork Server
---

最近做完IP工具，梳理下学到的知识

**关键词:whois UNIX FastCGI Nginx PHP GCC **

**CGI**全称是“公共网关接口”(Common Gateway Interface)，HTTP服务器与你的或其它机器上的程序进行“交谈”的一种工具，其程序须运行在网络服务器上。　CGI可以用任何一种语言编写，只要这种语言具有标准输入、输出和环境变量。如PHP,PERL等

### FASTCGI 特点

+ FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变。
+ FastCGI具有语言无关性

### FASTCGI 工作原理

+ Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)
+ FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。
+ 当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。
+ FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作

### Nginx+FastCGI运行原理

Nginx不支持对外部程序的直接调用或者解析，所有的外部程序（包括PHP）必须通过FastCGI接口来调用。FastCGI接口在Linux下是 socket（这个socket可以是文件socket，也可以是ip socket）。为了调用CGI程序，还需要一个FastCGI的wrapper（wrapper可以理解为用于启动另一个程序的程序），这个 wrapper绑定在某个固定socket上，如端口或者文件socket。当Nginx将CGI请求发送给这个socket的时候，通过FastCGI 接口，wrapper接收到请求，然后派生出一个新的线程，这个线程调用解释器或者外部程序处理脚本并读取返回数据；接着，wrapper再将返回的数据 通过FastCGI接口，沿着固定的socket传递给Nginx；最后，Nginx将返回的数据发送给客户端。这就是Nginx+FastCGI的整个 运作过程，如图1-3所示。

{% img /images/2013/fastcgi.jpg %}

### 我使用FastCGI的场景 

通过随机产生千万条IP地址，shell命令处理文本，用whois爬取信息，入库，接着用C语言写的客户端通过FASTCGI 实现与Nginx服务器的通信

####  通信方式

写好的c 程序在spawn-fcgi（一种fastcgi管理工具）下转换为unix sock ,由nginx指定文件监听sock文件描述符。

FastCGI 代码示例
```c
#include "fcgi_stdio.h"
int main(){
    int count = 0;
    while(FCGI_Accept() &gt;= 0) {
        //通信协议的格式，\r\n\r\n是向浏览器通信
        //，之前因为只写\r\n导致Page unavailable
        printf("Content-type: text/html\r\n"
                "\r\n"
                "Request number %d running on host <em>%s</em>\n",
                ++count, getenv("SERVER_NAME"));//环境变量
    }
    return 0;
}
```
### 感悟:

排查错误的基础是要有好的思维习惯，比如去看日志，然后定位错误。能让我写到linux c 程序很高兴,特别最后坚持写完字符串处理函数，同时也更认清自己的好多基础上的不足，在此感谢荣哥 :) 。

