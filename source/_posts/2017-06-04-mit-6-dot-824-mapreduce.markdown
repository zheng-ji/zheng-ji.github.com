---
layout: post
title: "MIT 6.824 MapReduce"
date: 2017-06-04 13:27
comments: true
categories: Programe
description: 分布式，MapReduce
---

学习完成 MIT 6.824 [Lab1 MapReduce](http://nil.csail.mit.edu/6.824/2017/labs/lab-1.html)，做下笔记

### MapReduce 的思路

* 把数据分成 M 份，每一份叫做 Mi
* 启动一个 master 对象，由它来控制如何分配调控
* master 挑出一个 worker，对 Mi 执行 map 操作，返回一个 KV 数组
* 然后把 KV 数组分成 nReduce 份存在本地，等待 Reduce 操作。当 map 全部完成后，每个 Mi 产生 nReduce 份结果，每一个叫做 Ri。文件名：mrtmp-JobName-Mi-Ri 其中Mi Ri 分别表示数字，因此这一步会产生 M * nReduce 份文件。
* 从每个 Mi 中选择一份 Ri。然后根据 Key 排序，把相同 Key 的 Value 合在一起，生成 Key /list(value)
* 开始 Reduce，输入list(value)，最后会生成 R 份文件 mrtmp.JobName-res-Ri
* 最后 Merge 成一个文件。

### 作业步骤

* Part1 完成 doMap 和 doReduce。doMap 完成3，4两个步骤. doReduce 完成5，6两个步骤。
* Part2 实现 main/wc.go 在Part1的基础上完成函数调用而已。
* Part3 把 map 和 reduce 的操作变成异步。用到了RPC，用channel 来实现并发控制。

### 代码笔记

[源码](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce)

* common.go [11-32行](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/common.go#L11)：可变参数打印日志，这个方法与C语言常用的类似
* common_reduce.go [Line75](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/common_reduce.go#L75)：sort.strings 对字符串切片排序
* commo_rpc.go [Line59](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/common_rpc.go#L59)：rpc调用方法
* master.go [Line95-99](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/master.go#L95)：当mr.newCond.Broadcast()被调用，此处就被唤醒，否则一直阻塞,mr.wait()所在的逻辑分支才会被唤醒，否则继续阻塞
* master.go [Line15-16](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/master.go#L15)：匿名参数，表示 Master 具有sync.Mutex的接口, 因而 Master 也能调用sync.Mutex的函数. 所以当调用 master.Lock()的时候也不足为奇
* master_rpc.go [Line14](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/master_rpc.go#L14),[Line37](https://github.com/zheng-ji/ToyCollection/blob/master/6.824-golabs-2017/src/mapreduce/master_rpc.go#L37)：chanel 被close的时候，case <- shutdown 也就被触发了 
