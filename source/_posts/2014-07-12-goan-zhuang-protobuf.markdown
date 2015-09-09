---
layout: post
title: "Go Protobuf"
date: 2014-07-12 17:39
comments: true
categories: Programe 
---

protobuf 是谷歌出品的，质量就无需质疑了，广泛运用于工业界
优点简而言之是：

* 二进制，速度快，便于拓展；
* 自动生成代码接口，支持多语言

对于第一点, 貌似优秀的开源代码没这么干都不好意思开放。

### 安装

比较蛋疼,折腾一小段时间。

```
go get code.google.com/p/goprotobuf/{proto, protoc-gen-go}

go install code.google.com/p/goprotobuf/proto

sudo apt-get install protobuf-compiler
```

### 使用

是在项目内部新建一个proto.文件, 然后执行

```
protoc --go_out=. xxx.proto
```

