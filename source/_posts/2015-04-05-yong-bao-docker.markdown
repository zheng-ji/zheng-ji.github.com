---
layout: post
title: "为服务端程序构建docker"
date: 2015-04-05 20:24
comments: true
categories: Server
description: Docker, 自动化运维
---

Docker 的优点自从问世就一直被工业界热论。

平时工作中，所部署的大多数`Python`项目都会用上 [virtualenv](http://wiki.zheng-ji.info/Python/virtualenv-py.html), 
沙箱的隔离带来的好处自然不言而喻。我也希望静态编译的服务，比如 `Golang` `C++` 的项目
同样能使用上独立的沙箱环境。得益于`Docker`，我们仍然可以做到。


这个过程没有想象中的简单，需要经过一番折腾，我用最近写的 KafkServer 为例，叙述我是怎么构建的，需要读者具备一定的 Docker 基础.


### 一览该 Docker 项目

```
zj@zheng-ji:~/workspace/gocode/src/kafconsumer/docker$ tree
.
├── Dockerfile
├── kafConsumer
│   ├── consumer
│   ├── etc
│   │   ├── config.yml
│   │   └── logger.xml
│   └── script
│       └── start.sh
└── kafConsumer.tar.gz
```

以上的截图，是一个完整的 `Docker` 项目，包含了：

* `Dockerfile`,
* `kafCounsumer`(服务端程序，里面附带的启动脚本，配置程序，以及二进制文件)，
* 还有它被压缩而成的 `kafConsumer.tar.gz`

---

### Dockerfile 的内容


```
FROM ubuntu:14.04                                                         
MAINTAINER zheng-ji <zheng-ji.info>                                     
RUN echo Asia/Shanghai > /etc/timezone                   
RUN sed -i "s/archive\.ubuntu/mirrors.163/" /etc/apt/sources.list          
RUN apt-get update                                                         
COPY kafConsumer.tar.gz /                                                  
RUN tar xvf kafConsumer.tar.gz                                         
VOLUME /data                   
WORKDIR /kafConsumer                                                   
ENTRYPOINT ["./script/start.sh"]
```

`Dockerfile` 可以理解为`makefile` 之类的文件，Docker 可以依照文件中的内容，构建镜像.

```
sudo docker -t build Server/KafConsumer .
```

这样就生成了`Tag` 为 `Server/KafConsumer` 的镜像，待会儿我们会使用它


以上 `Dockerfile` 的具体内容的意义是:

* 第一行：拉取ubuntu 14:04的镜像源
* 第二行：维护者
* 第三行：调整时区
* 第四行：更新源地址
* 第五行：更新源
* 第六行：复制项目下的压缩包到虚拟机根目录
* 第七行：解压
* 第八行：项目中使用/data数据卷
* 第九行：进入工作目录
* 第十行：Docker的入口执行文件是start.sh

---


### 入口文件的内容

```
#!/bin/bash
ulimit -a
if [ ! -d /data/ad ];  then
    mkdir /data/ad
fi
exec ./consumer -c=etc/config.yml
```

这是一个shell的启动文件，因此一定要在开头写明 #!/bin/bash,
使用exec 执行程序


---

### 启动镜像

```
sudo docker run -i -t  -v /path/to/data:/data Server/kafConsumer
```
这样就执行了，-v 可以映射你的本地文件到虚拟机的某个数据卷，这样我们就能从外面看到程序产生的文件.

###如果你想关闭或者重启该服务的怎么办

```
sudo docker ps -a

找到你的 Docker 容器

CONTAINER ID    IMAGE           COMMAND                CREATED        STATUS        PORTS    NAMES
5b39d0d5cb85    Server/kafkaconsumer:latest   "./script/start.sh"    3 hours ago    tender_bohr 
```

启动或者关闭

```
sudo docker start tender_bohr
sudo docker stop tender_bohr
```

---

### Daocloud  加速

功夫墙的原因，国外很多镜像被墙，因此构建镜像很慢，使用 Daocloud 服务可以加速,注册后就有该服务了

```
cat /etc/default/docker
DOCKER_OPTS="$DOCKER_OPTS --registry-mirror=http://xxxxxx.m.daocloud.io"
```
