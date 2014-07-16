---
layout: post
title: "折腾docker"
date: 2014-07-16 20:20
comments: true
categories: Server 
keywords: ubuntu docker
description: ubuntu docker
---

  早在1年前就听过 [docker](http://docs.docker.com/)这个 Golang 社区的明星产品 。

### 简单地介绍

  Docker 提供了一个可以运行你的应用程序的容器。像一个可移植的容器引擎那样工作。它把应用程序及所有程序的依赖环境打包到一个虚拟容器中，这个虚拟容器可以运行在任何一种 Linux 服务器上。这大大地提高了程序运行的灵活性和可移植性，极大的降低运维成本。

### 组成

* Docker 服务器守护程序（server daemon），用于管理所有的容器。
* Docker 命令行客户端，用于控制服务器守护程序。
* Docker 镜像：查找和浏览 docker 容器镜像。它也访问这里得到：[链接](https://index.docker.io/)


### 有了虚拟机为什么还要docker?
  virtualbox 等虚拟机提供的是完整的操作系统环境, 迁移的时候太大了。它们包含了大量类似硬件驱动、虚拟处理器、网络接口等等并不需要的信息，也需要比较长时间的启动，同时也会消耗大量的内存、CPU 资源。

  Docker 相比起来就非常轻量级了。运行起来就和一个常规程序差不多。这个容器不仅仅运行快，创建一个镜像和制作文件系统快照也很快,甚至比vagrant更节约资源

### 初体验
下载docker 并安装 ubuntu 12.04 这里如果没有指明 `ubuntu:12.04`, 会将所有ubuntu镜像都下载。由于被墙，速度惨不忍睹.

```
zj@zheng-ji:~$ sudo apt-get install docker.io
zj@zheng-ji:~$ sudo docker pull ubuntu:12.04 
```

查看已有的镜像

```
zj@zheng-ji:~$ sudo docker images
```

使用 bash 命令进入docker的指定镜像

```
zj@zheng-ji:~$ sudo docker run -t -i -p 3000 ubuntu:12.04 /bin/bash
```

这时候我就可以像一个新系统一样把玩了, 比如创建一个目录。

```
zj@zheng-ji:~$ mkdir /home/zj/test
```

好了，我想把这个现场保存下来做移植到别的地方直接使用。需要到 [这里]（https://registry.hub.docker.com/u/）注册一个账户，然后再上传，类似在 github  上面一样的操作。

```
sudo docker ps 

//因为我已经commit 过一次，所以名字变成我的别名 zhengji/helloworld
zj@zheng-ji:~$ docker ps
CONTAINER ID        IMAGE                       COMMAND             CREATED             STATUS              PORTS                     NAMES
8dad3aa4451f        zhengji/helloworld:latest   bin/bash            27 minutes ago      Up 27 minutes       0.0.0.0:49156->3000/tcp   high_galileo      
```

可以看到要保存的镜像的 CONTAINER ID

这个时候提交

```
zj@zheng-ji:~$ sudo docker commit 8dad3aa4451f zhengji/helloworld -m "test"
61f9527f368395ee228af07062b3f1a26fa7143ba2721c4f9755ae93d588358e

zj@zheng-ji:~$ sudo docker push zhengji/helloworld
```

下次pull 下来就可以使用了

```
sudo docker pull zhengji/helloworld
```

----------

### 可能遇到的坑

* 需要编辑 /etc/default/docker.io 然后编辑里面的,使得其可以解决 DNS解析

```
DOCKER_OPTS = "-dns 8.8.8.8"
```

* 设置 ufw 端口可转发

```
vim /etc/default/ufw
DEFAULT_FORWARD_POLICY = "ACCEPT"
```


### 参考链接

* [docker 中文](http://www.docker.org.cn/book/docker.html)
* [docker官网](http://www.docker.org.cn/book/docker.html)
* [Docker入门教程](http://segmentfault.com/a/1190000000366923)


