---
layout: post
title: "制作setup.py"
date: 2014-05-03 11:52
comments: true
categories: Programe
---
做好一个python module 之后，可以把源代码包含进项目中。OK，但是如果有很多依赖项目的话就不好管理了，这个时候我们用 setuptools 工具来安装到系统的包。

以下给出一个 demo 

```
mkdir hacker
vim foo.py
```

编辑以下内容

```
class Hacker():
    def __init__(self):
        self.motto = "No evil, Be Hacker"
    def printMotto(self):
        print self.motto
```

开始编写 setup.py, 也许是为了知识产权保护，里面按照规矩会让作者填上资料。

```
from distutils.core import setup
setup(name='Hacker',
        version='1.0',
        description='be hake or be hacked',
        author='zheng-ji',
        author_email='zhengji91@gmail.com',
        url='http://zheng-ji.info',
        py_modules=['foo'],
        install_requires=["requests"]
     )
```

好了，这个时候执行,

```
sudo python setup.py install
```

该命令，会把模块复制到 `/usr/lib/python2.7/` 下面. 之后可以直接 import foo, 使用该模块。

删除的话，就得蛋疼去 `/usr/lib/python2.7/` 手动删除,在 mac 里面是 `/Library/Python/2.7/site-packages`






