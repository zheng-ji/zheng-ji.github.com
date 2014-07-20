---
layout: post
title: "TrieTree 中文分词"
date: 2012-05-26 23:45
comments: true
categories: Programe
---
**Trie树**，即字典树，又称单词查找树或键树，是一种树形结构，是一种哈希树的变种。典型应用是用于统计和排序大量的字符串（但不仅限于字符串），所以经常被搜索引擎系统用于文本词频统计。它的优点是：最大限度地减少无谓的字符串比较，查询效率比哈希表高。
Trie的核心思想是空间换时间。利用字符串的公共前缀来降低查询时间的开销以达到提高效率的目的。

它有3个基本性质：

+ 根节点不包含字符，除根节点外每一个节点都只包含一个字符。
+ 从根节点到某一节点，路径上经过的字符连接起来，为该节点对应的字符串。
+ 每个节点的所有子节点包含的字符都不相同。

```python
class Node:
    def __init__(self,char,is_end=False):
        self.is_end = is_end
        self.prefix_count = 0
        self.char = char
        self.children = [None for child in range(ALPHABET_SIZE)]

class Trie:
    def __init__(self):
        self.head = Node('')

    def insert(self, word):
        current = self.head
        count = 0
        for letter in word:
            int_value = ord(letter) - ord('a')
            try:
            assert ( 0 <= int_value <= 26)
            except Exception:
                raise Exception("invalid Word")
            if current.children[int_value] is None:
               current.children[int_value] = Node(letter)
               current = current.children[int_value]
               count += 1
               current.prefix_count = count
            else:
                current = current.children [int_value]
        current.is_end = True

    def search(self, word):
        current = self.head
        for letter in word:
            int_value = ord(letter) - ord('a')
            try:
                assert (0 <= int_value >= 26)
            except Exception:
                return False
            if current.children[int_value] is None:
                return False
            else:
                current = current.children[int_value]
        return current.is_end

    def count_words_with_prefix(self, word):
        current = self.head
        for letter in word:
            int_value = ord(letter) - ord('a')
            try:
                assert (0 <= int_value <= 26)
            except Exception:
                return None
            if current.children[int_value] is None:
                return None
            else:
                current =  current.children[int_value]
        return current.prefix_count         

if __name__ == '__main__':
    file = open('dict-test.txt','r')
    dicts =  file.read().split()
    t = Trie()
    for item in dicts:
    t.insert(item)
    print t.search('c')
    print t.search('love')
    print t.search('c++')
    raw_input('void flush')__init__(self,char
```

附带go 语言的:

```
package main

import ("fmt")

type Node struct {
    is_end           bool
    element          byte
    prefix_count     int
    children         [26]*Node
}

type Trie struct {
    head *Node
}

func (trie *Trie) insert(word string) {
    current := trie.head
    count := 0
    for _, v := range word {
        index := v - 'a'
        if nil == current.children[index] {
            current.children[index] = new(Node)
            current = current.children[index]
            count += 1
            current.prefix_count = count
        } else {
            current = current.children[index]
        }
    }
    current.is_end = true
}

func (trie *Trie) search(word string) bool {
    current := trie.head
    for _, v:= range word {
        index := v - 'a'
        if nil == current.children[index] {
            return false
        } else {
            current = current.children[index]
        }
    }
    return current.is_end
}

func main() {
    trie := new(Trie)
    trie.head = new(Node)
    if nil != trie.head.children[5] {
        fmt.Println("element is not null")
    }
    if nil == trie.head.children[5] {
        fmt.Println("element is null")
    }
    trie.insert("hello")
    trie.insert("world")
    if true == trie.search("hell") {
        fmt.Println("good")
    }
}
```


#### Node类：

+ is_end 用于标识该结点是否是一个单词结尾，默认为False
+ prefix_count: 该字符前缀有几个字符
+ char:该节点带有字符
+ children:该节点的子结点集合（这里有26个默认的NONE结点）
 
####  Trie类：
+ head 是头结点，默认为空结点
+ 函数：insert(self,word):插入字符，创建结点，如果该单词到达最后一个字母就设置is_end=True
+ search(self, word)：查询单词是否存在，是返回True
+ count_words_with_prefix(self, word)：查询结点的前缀
