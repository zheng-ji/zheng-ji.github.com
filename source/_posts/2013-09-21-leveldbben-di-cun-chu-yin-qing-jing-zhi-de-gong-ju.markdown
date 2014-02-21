---
layout: post
title: "leveldb本地存储引擎-精致的工具"
date: 2013-09-21 09:42
comments: true
categories: Server DataBase 
---

 leveldb是一个google实现的非常高效的kv数据库，能够支持billion级别的数据量了。 在这个数量级别下还有着非常高的性能，主要归功于它的良好的设计。特别是LSM算法。 LevelDB 是单进程的服务，性能非常之高，在一台4个Q6600的CPU机器上，每秒钟写数据超过40w，而随机读的性能每秒钟超过10w。@ideawu的ssdb就是基于Leveldb引擎开发的，看起来很NB的样子 

它只是一个本地存储引擎:

* k/v db library，提供持久化 
* No Server 
* No cache 
于是它精巧。 

>talk is cheap ,Show me the Code

简单示范一下C++使用Leveldb ,要先下载leveldb源码编译[code](https://github.com/basho/leveldb)

```python
#include<iostream>  
#include<string>  
#include"leveldb/db.h"  

using namespace std;  
using namespace leveldb;  

int main()  
{  
    DB *db;  
    Options options;  
    options.create_if_missing = true;  
    Status s = DB::Open(options,"/data/leveldb/lvd.db",&db);  
    if(!s.ok()){  
        cerr << s.ToString() << endl;
    }  
    string key = "name",val = "leveldb";  
    s = db->Put(WriteOptions(),key,val);  
    if(!s.ok()){  
        cerr << s.ToString() << endl; 
    }  
    s = db->Get(ReadOptions(),key,&val);  
    if(s.ok()){  
        cout << key << "=" << val << endl;  
        s = db->Put(leveldb::WriteOptions(),"key2",val);  
        if(s.ok()){  
            s = db->Delete(leveldb::WriteOptions(),key);  
            if(!s.ok()){  
                cerr << s.ToString() << endl;
            }  
        }  
    }  
                                                                                                                                    }  
```

编译方法

```
g++ testleveldb.cpp -o main -I./Include -l./ -lleveldb -lrt
```

执行./main 之后便可以看到,leveldb的存储文件

```
zj@hp:~/lvd.db$ tree
.
├── 000003.log
├── CURRENT
├── LOCK
├── LOG
├── MANIFEST-000002
├── sst_0
├── sst_1
├── sst_2
├── sst_3
├── sst_4
├── sst_5
└── sst_6
```

很多语言都给予了binding,那么python 也不例外了 ：） [py-leveldb](https://github.com/rjpower/py-leveldb)  
你需要先安装python-leveldb库

```python
import leveldb
db = leveldb.LevelDB('./db')
db.Put('hello', 'world')
print db.Get('hello')
```

如果你也和我一样蛋疼，你会不会想会有Server可以调用呢，再看下一篇吧 ：）






