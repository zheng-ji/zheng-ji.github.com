---
layout: post
title: "mac 安装Go"
date: 2014-05-03 09:32
comments: true
categories: Programe
---
之前是在 Ubuntu 配置Go 的环境，换了 Mac 后，也来折腾一番，其实很简单。

修改 `~.zshrc`,添加以下环境变量

```
export GOROOT=$HOME/go
export GOBIN=$GOROOT/bin
export PATH=$PATH:$GOBIN
```


开始安装,耗时有点久
保证你的电脑安装有 hg, 如果没有,请执行 

```
sudo easy_install mercurial
```

```
hg clone -u release https://code.google.com/p/go
cd go/src
./all.bash
```

开始检验, 执行 go , 出现一系列 Usage信息就算通过了 ：）。

