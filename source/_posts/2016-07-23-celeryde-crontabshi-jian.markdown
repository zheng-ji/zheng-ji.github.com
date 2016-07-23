---
layout: post
title: "Celery的Crontab实践"
date: 2016-07-23 20:19
comments: true
categories: Programe
description: celery crontab
---

有时候我们需要处理耗时的操作，同时又要保持较快的响应速度，就需要借助异步队列的帮助。Celery 作为异步队列服务，想必是很多人和我一样的选择。用法在官方文档也详细介绍，不再赘述。
  
这次想记录的是用 Celery 来实现定时任务。这里也有一点点坑。
  
以下是 `main.py` 的内容
 
```py
from celery import Celery
from lib import distribute
from celery.schedules import crontab

app = distribute.app
app.conf.update(
    CELERYBEAT_SCHEDULE = {
        'every-minute': {
            'task': 'test_cron',
            'schedule': crontab(minute="*"),
            'args': (16, 13),
        }
    },
    CELERY_INCLUDE=("apps.tasks",)
)
if __name__ == '__main__':
    app.start()
```

实际工作单元,我放在 apps 目录下的 `tasks.py` 文件中

```py
from lib.distribute import app
@app.task(name="test_cron")
def mul(x, y):
    return x * y
```

上述是一个简单的 Crontab 应用，它仅需要以下命令就能执行,
其中  `--beat` 表示 crontab 的应用

```
python main.py worker --beat -l info
```

起初我想把异步队列和定时任务放在一起,就加上了一句 CELERY_QUEUES 的配置

```py
app.conf.update(
    // 添加的部分
    CELERY_QUEUES=(
        Queue(
          'test', Exchange('test_exchange'),
           routing_key='test_queue'
        ),
    ),
    CELERYBEAT_SCHEDULE = {
        'every-minute': {
            'task': 'test_cron',
            'schedule': crontab(minute="*"),
            'args': (16, 13),
        }
    },
    CELERY_INCLUDE=("apps.tasks",)
)
```

同样用上述命令开启worker，发现这个时候 Crontab 不能工作了，后来看到官方的文档：

> celery beat and celery worker as separate services instead. 

也就是说 Celery 的 Beat 需要和其他异步worker 分开，单独执行。

相关代码[链接](https://github.com/zheng-ji/ToyCollection/tree/master/celery_proj)
