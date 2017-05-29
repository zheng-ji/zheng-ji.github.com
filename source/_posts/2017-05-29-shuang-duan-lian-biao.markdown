---
layout: post
title: "双端链表"
date: 2017-05-29 23:19
comments: true
categories: Programe
description: 双端链表
---

Nginx, Redis 项目中，均使用上了双链表。

* 列表类型的键进行操作(RPUSH, LPOP)时，程序底层操作用的是双端链表。 [源码](https://github.com/antirez/redis/blob/unstable/src/adlist.c)

* Nginx 典型应用是在连接池中。Nginx 会处理大量的 socket 连接，为提高并发处理链接的能力，引入了连接池，其实现这个连接池用到了双链表。[源码](https://github.com/nginx/nginx/blob/master/src/core/ngx_queue.c).

对于双端链表，教科书上曾有提及，但如今映像并不深刻，再度理解并实践一次。

### 结构定义与初始化

```
typedef struct Node {
    int num;
    struct Node * next; //前继指针
    struct Node * pre;  //后继指针
} Node;

typedef struct dlist {
    Node * head; //双链表的指向头结点的元素
    Node * tail; //双链表指向尾部节点的元素，方便从后检索
} dlist;
```

初始化的双链表，`head`,`tail` 均为空。

### 插入操作

插入操作，是按照递增的顺序插入，涉及到双链表的更改，因此传参是指向链表的指针，分三种情况

* 空链表插入头元素，head 与 tail 节点均要修改
* 在链表尾部插入节点，要变动 tail 指针
* 中间插入节点

{% img /images/2017/05/dlist-insert.png %}

```
static int insertDlist(dlist **list, int num) {
    Node * head = (*list) -> head;
    Node * tail = (*list) -> tail;

    if (list == NULL) return -1;

    Node * node = initNode(num);
    if (node == NULL) return -1;

    // empty dlist
    if ((*list) -> head == NULL && (*list) -> tail == NULL) {
        (*list)-> head = node;
        (*list)-> tail = node;
        return 0;
    }

    while (head -> next && head -> num < num) {
        head = head -> next;
    }

    // at the end
    if (head->next == NULL) {
        head -> next = node;
        node -> pre = head;
        tail -> pre = node;
        return 0;
    } else {
        // in the middle
        node-> next = head -> next;
        head -> next -> pre = node;
        node -> pre = head;
        head -> next = node;
        return 0;
    }
}

```

### 删除操作

删除操作，要涉及到双链表的更改，因此传参是指向链表的指针，分三种情况

* 删除头结点，
* 删除尾节点
* 删除中间节点

{% img /images/2017/05/dlist-delete.png %}

```
static int deleteDlist(dlist ** list, int location) {
    Node * head = (*list) -> head;
    Node * now = NULL;
    Node * last = NULL;

    if (head == NULL)
        return -1;
    if (location <= 0 || location > numOfDlist(*list))
        return -1;

    if (location == 1) {
        now = head;
        (*list) -> head = now ->next;
        head -> next ->pre = NULL;
        if (now) {
            free(now);
            now = NULL;
        }
        return 0;
    }
    int num = 0;
    while (head && num++ < location) {
        head = head -> next;
    }

    if (head -> next == NULL) {
        now = (*list) -> tail;
        head -> pre -> next = NULL;
        now -> pre = head->pre;
        if (head) {
            free(head);
            head = NULL;
        }
    } else {
        now = head -> next;
        last = head -> pre;
        now ->pre = head -> pre;
        last -> next = head ->next;
        if (head) {
            free(head);
            head = NULL;
        }
    }
    return 0;
}
```

本文所述是 C 语言实现的，[源码](https://github.com/zheng-ji/ToyCollection/dlist/mydlist.c)
在 Go 语言之中, `container/list` 包实现了双链表，直接引入就可以使用了。
