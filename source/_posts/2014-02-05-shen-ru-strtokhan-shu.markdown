---
layout: post
title: "深入strtok函数"
date: 2014-02-05 16:35
comments: true
categories: Programe 
keywords: strtok
description: strtok源码
---

下午在写代码的时候用到了 strtok , 曾经是用过该函数的, 但每次书写都需要翻阅 C++ manual,而且需要多次调试, 每次需要用的时候速查, 看似解决问题，下次需要的时候又忘掉了。反而浪费时间遂决心去阅读源码。

### Man strtok
```c
strtok的函数原型为:
char *strtok(char *s, char *delim)
```
功能以包含在delim中的字符为分界符，将s切分成一个个子串；
如果s为空值NULL，则函数保存的指针 SAVE_PTR在下一次调用中将作为起始位置，
这里说到的save_ptr是啥？还未知，
但可以知道s在初次调用时为原始字符串，获取第一个分割得到的字符串，
之后传进的参数为NULL，逐次获取字符串，当返回值为NULL的时候，解析该字符串结束

### How to use?
```c
char s[100] = " zheng-ji is so stupid";
char *del = " ";
char *token ;
token = strtok(s, del);
while(token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, del);
}
```

输出：

```
zheng-ji
is
so
stupid
```

###探究save_ptr的获得的结论

```c
static char *olds;
char * strtok (char *s，const char *delim）{
    char *token;
    if (s == NULL) s = olds;
    /*将指针移到第一个非delim中的字符的位置*/
    s += strspn (s, delim); 
    if (*s == '\0') {
        olds = s;
        return NULL;
    }
    /*获取到delim中字符在字符串s中第一次出现的位置*/
    token = s;
    s = strpbrk (token, delim);
    if (s == NULL)
        olds = __rawmemchr (token, '\0');
    else {
        /*重新给old赋值 */
        *s = '\0';
        olds = s + 1;
    }
    return token;
}
```

源码告知我们:

+ 之前我们觉得神奇的SAVE_PTR(源码中为 static char * olds)原来是个静态字符指针，每次调用都被赋值。
+ 如果第一位出现了需要被删除的字符，是会被跳过的。
+ 如果传入的s为NULL，则将olds 作为继续解析的字符串。
+ 该函数会修改原有的字符串。
+ 非线程安全的函数，由于用到了静态全局变量，因此当有多个线程同时调用这个函数的时候就会出现问题.这个静态的指针变量就会变的混乱。同时在同一个程序中同时有两个字符串要解析，并且同时进行解析也是会出错的。
