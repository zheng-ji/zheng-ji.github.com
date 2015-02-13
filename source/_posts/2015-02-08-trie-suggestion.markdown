---
layout: post
title: "实现一个智能提示框功能"
date: 2015-02-08 13:40
comments: true
description: golang trie
categories: Programe
---

耗了3个夜晚出来的东西.

先上图吧：

{% img /images/2015/02/suggest.png %}

是的，就是这样一个类似百度框的联想提升功能，让我纠结了几个晚上，在实现成功的那一瞬间，着实感受到编程之美。很享受为了一个小idea折腾的酣畅淋漓的过程。

起因: 在公司`GitLab`看到有这人对内部管理系统提出了这个需求，但是一直没有被`close`, 我觉得应该挺有趣的，好奇心驱动下就开始搞了。

如果仅仅是实现这个需求，应该有很多种

* 方法一：使用一个hash,将关键字填入key，如果采用此法，数据量大的时候估计堪忧，以一个汉字2个Byte计算的话，1kw个词条，1个词条10个词语的话需要占用大于200M内存。
* 方法二：前缀匹配，那么应该怎么选择数据结构,朴素的做法是O(N^N),我们肯定采用复杂度较优的Trie树  O(1)
* 方法三：Radix Tree 据说这个是linux cache的一个算法。看了几个小时，真心复杂，不得不佩服内核开发者！


### 讲一讲`trie`树吧

{% img /images/2015/02/triestruct.png %}

* 根节点不包含字符，除根节点外的每一个子节点都包含一个字符。
* 根节点到某一节点，路径上经过的字符连接起来，就是该节点对应的字符串。
* 每个单词的公共前缀作为一个字符节点保存。
* 叶子节点的指针是空的

以下是具体的数据结构代码

```
type Node struct {
    Link     map[string]*Node  #指针
    Key      string                     #每个节点的字符
    IsLeaf   bool                       #是否叶子节点 
    Weight   float64                    #权重
    LongWord string                     #从根节点到该节点的长字符
}
```

围绕这个数据结构做了
构建树，删除节点，添加节点，获取子节点等

### 知识铺垫：
在用 Go 语言处理中文字符的时候，需要特别使用 []rune数组，看以下示范代码就知道了,他把中文处理成1个字符表现的编码方式了。正式我们下列处理Trie需要用到的。

```
package main
import "fmt"

func main () {
    m_str := "编程"
    fmt.Println("fmt:", m_str)
    m_str_rune := []rune(m_str)
    fmt.Println("fmt:", m_str_rune)
    m_str_byte := []byte(m_str)
    fmt.Println("fmt:", m_str_byte)
}

$ ./test_rune
fmt: 编程
fmt: [32534 31243]
fmt: [231 188 150 231 168 139]
```

### 测试结果：
导入100W条词条,搜索的反应是瞬秒，1ms返回响应，在4G的机器上，整个程序占用内存0.3%。

---


每个成熟的互联网产品，背后都是工程师耗费一点一滴思维的结晶构建而成的。对待技术不得不敬畏。

[代码](https://github.com/zheng-ji/trietips)
