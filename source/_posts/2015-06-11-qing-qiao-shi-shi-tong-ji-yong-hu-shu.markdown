---
layout: post
title: "轻巧实时统计用户数"
date: 2015-06-11 23:14
comments: true
categories: Programe
description: Redis bitset
---

## 背景 ##

最近在优化一个短地址的统计服务，之前是使用 Cookie 来做统计每天的UV，而且这个需求是近乎实时的，
业务方需要每5分钟就能看到最新统计结果。但有些情况我们是取不到Cookie的，比如服务器对服务器的狂刷访问，那么UV就计算不准确，
是时候要改造方案了。

后来我用 IP+UserAgent 来识别用户，从而统计 UV。好了，接下来你会怎么做这个实时统计呢？

##两个方案的选择##

* Plan A：

将每天的 `IP+UA` 存进 Redis 的 Set 集合里，它会自动去重，然后计算该集合里元素的个数得到结果，此方案似乎不错，
但真的好吗？假如每天大概有200W个UV，1个用户标识`IP+UA`需要大概150个字节，那么大约要耗费300MB的内存。

觉得内存太宝贵，应该有更好的方法，想起了位运算， 于是就有了

* Plan B:

将 `IP+UA` 组合成的字符串哈希成一个数值，然后借助 Redis 的 BitSet 数据结构求出`UV`。以下是伪代码

```
conn = redis()
index = hash(UA+IP)
key = "xxx_2015-05-10"
conn.do('SETBIT', key, index, 1) # 将该hash值对应的位赋值为1

realtime_uv = conn.do('BITCOUNT', key) # 得到实时的uv
```

根据业务的情况，我的hash桶开了200W个位，大概需要消耗2M的内存，的确节约不少空间，位运算的效率也很快。
于是欣然选择 Plan B


## 一些链接 ##

* [Redis Set](https://redis.readthedocs.org/en/2.4/set.html)
* [Redis BitSet](http://redis.io/commands/SETBIT)
* [NoSQLFan 一个文章](http://blog.nosqlfan.com/html/3501.html)

