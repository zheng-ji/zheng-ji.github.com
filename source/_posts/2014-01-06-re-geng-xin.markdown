---
layout: post
title: "热更新"
date: 2014-01-06 20:58
comments: true
categories: Programe
---
  Server 端程序很多时候需要修改配置，这个时候如果重启服务就显得很不友好，代价也高。Nginx和Django，采用的是平滑启动.自动热更新,折腾的东西中遇到这样一个诉求，用python实现的话:
 
1. 当修改配置后,通过对比py的修改时间，来推断该模块是否被修改过,从而reload指定模块就可以在不重启服务的情况下读取修改的配置,
2. sys.module['modname']返回的是该模块的pyc文件，而该文件是没有修改时间的，所以计算修改时间需要取py文件的属性

```python
import time
import sys, os
def auto_reload():
    mods = ["test_config"]
    for mod in mods:
        try:
            module = sys.modules[mod]
        except:
            continue
        filename = module.__file__
        print filename
        if filename.endswith(".pyc"):
            filename = filename.replace(".pyc", ".py")
            mod_time = os.path.getmtime(filename)
        if not "loadtime" in module.__dict__:
            module.loadtime = 0 # first load's time  1*
            try:
                if mod_time > module.loadtime:
                    reload(module)
                except:
                    pass
        module.loadtime = mod_time # 2*

if __name__ == "__main__":
    import time
    import my_config
    tmp = None
    while True:
        auto_reload()
        if test_config.address != tmp:
        print test_config.address
        tmp = test_config.address
        time.sleep(2)
```

test_config.py

```
address = "127.0.0.1"
```
