---
layout: post
title: "Thread Pool in Python"
date: 2012-12-27 13:40
comments: true
categories: Programe
---
#### python的创建多线程的方法
使用线程有两种模式：
+ 一种是创建线程要执行的函数，把这个函数传递进Thread对象里，让它来执行；
+ 一种是直接从Thread继承，创建一个新的class，把线程执行的代码放到这个新的 class里。

```python
import string, threading, time
 
def thread_main(a):
        global count, mutex
            # 获得线程名
            threadname = threading.currentThread().getName()
 
    for x in xrange(0, int(a)):
                # 取得锁
                mutex.acquire()
        count = count + 1
                # 释放锁
                mutex.release()
        print threadname, x, count
                time.sleep(1)
 
def main(num):
    global count, mutex
    threads = []
         
    count = 1
    # 创建一个锁
    mutex = threading.Lock()
    # 先创建线程对象
    for x in xrange(0, num):
        threads.append(threading.Thread(target=thread_main, args=(10,)))
    # 启动所有线程
    for t in threads:
        t.start()
    # 主线程中等待所有子线程退出
    for t in threads:
        t.join() 
if __name__ == '__main__':
    num = 4
    # 创建4个线程
    main(4)
```
方法二

```python
import threading
import time
 
class Test(threading.Thread):
    def __init__(self, num):
        threading.Thread.__init__(self)
        self._run_num = num

    def run(self):
        global count, mutex
        threadname = threading.currentThread().getName()

    for x in xrange(0, int(self._run_num)):
        mutex.acquire()
        count = count + 1
        mutex.release()
        print threadname, x, count
        time.sleep(1)

if __name__ == '__main__':
    global count, mutex
    threads = []
    num = 4
    count = 1
    # 创建锁
    mutex = threading.Lock()
    # 创建线程对象
    for x in xrange(0, num):
        threads.append(Test(10))
    # 启动线程
    for t in threads:
        t.start()
    # 等待子线程结束
    for t in threads:
        t.join()
```
#### 队列同步
Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步

#### 线程池原理：
我们把任务放进队列中去，然后开N个线程，每个线程都去队列中取一个任务，执行完了之后告诉系统说我执行完了，然后接着去队列中取下一个任务，直至队列中所有任务取空，退出线程.

```python
import time
import threading
import Queue
 
class Worker(threading.Thread):
    def __init__(self, name, queue):
        threading.Thread.__init__(self)
        self.queue = queue
        self.start()
    def run(self):
        # 著名的死循环，保证接着跑下一个任务
        while True:
        # 队列为空则退出线程
        if self.queue.empty():
            break
        # 获取一个项目
        foo = self.queue.get()
        # 延时1S模拟你要做的事情
        time.sleep(1)
        print self.getName(),':', foo
        # 告诉系统说任务完成
        self.queue.task_done()
 
queue = Queue.Queue()
 
# 加入100个任务队列
for i in range(100):
    queue.put(i)
             
# 开10个线程
for i in range(10):
    threadName = 'Thread' + str(i)
    Worker(threadName, queue)
# 所有线程执行完毕后关闭
queue.join()
```


