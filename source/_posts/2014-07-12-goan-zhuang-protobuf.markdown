---
layout: post
title: "go安装protobuf"
date: 2014-07-12 17:39
comments: true
categories: Programe 
---

protobuf是谷歌出品的，质量就无需质疑了，广泛运用于工业界
优点简而言之是：

* 二进制，速度快，便于拓展；
* 自动生成代码接口，支持多语言

对于第一点, 貌似优秀的开源代码没这么干都不好意思开放。

关于 `protobuf` 的使用方法网上资料丰富。记录下使用 golang 玩protobuf的东西

安装, 比较蛋疼,折腾一小段时间。

```
go get code.google.com/p/goprotobuf/{proto, protoc-gen-go}

go install code.google.com/p/goprotobuf/proto

sudo apt-get install protobuf-compiler
```

使用的时, 是在项目内部新建一个proto.文件, 然后执行

```
protoc --go_out=. xxx.proto
```

