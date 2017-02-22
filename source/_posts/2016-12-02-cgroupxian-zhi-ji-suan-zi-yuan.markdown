---
layout: post
title: "cgroup 限制计算资源"
date: 2016-12-02 15:41
comments: true
categories: System
description: cgroup
---

Cgroup 实现了对计算机资源使用上的隔离，它是 Docker 底层的基础技术。我们可以用它来限制程序使用的CPU、内存、磁盘。

### 安装

在 Ubuntu 14.04 下安装的方法：

```
sudo apt-get install cgroup-bin
```

安装完后执行
`mount -t cgroup` 会出现如下，可以看到它其实是一个文件系统

```
cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,relatime,cpuset)
cgroup on /sys/fs/cgroup/cpu type cgroup (rw,relatime,cpu)
...
cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,relatime,hugetlb)
```

如果没有看到以上的目录，这时候需要手动 mount 了

```
cd /sys/fs
mkdir cgroup
mount -t tmpfs cgroup_root ./cgroup
mkdir cgroup/cpuset
mount -t cgroup -ocpuset cpuset ./cgroup/cpuset/
mkdir cgroup/cpu
mount -t cgroup -ocpu cpu ./cgroup/cpu/
mkdir cgroup/memory
mount -t cgroup -omemory memory ./cgroup/memory/
```

### 实践

我们来感性认识下 cgroup 吧，编写一个耗费 CPU 的程序，姑且叫暴走程序(baozou)

```py
count = 0
while True:
    count = count + 1 - 1
```

运行该程序，top -p 之，100% CPU使用率

```
  PID      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                       
16515      20   0    2336    728    544 R  100.0  0.0   1:27.23 baozou
```

我想限制暴走程序 CPU 使用该如何做? 我们手动创建一个 cgroup 目录来针对它。

```sh
cd /sys/fs/cgroup/cpu
mkdir calm      // 名字可自定义
ls /calm        // 该目录下自动生产与 CPU 有关的文件
cgroup.clone_children  cpu.cfs_period_us  cpu.shares  notify_on_release
cgroup.procs           cpu.cfs_quota_us   cpu.stat    tasks
```

接着写入限制规则

```
// 默认是100000，20000意味着限制它的cpu为20%
echo 20000 > /sys/fs/cgroup/cpu/calm/cpu.cfs_quota_us, 

// 写入程序的 PID 16515
echo 16515 > /sys/fs/cgroup/cpu/calm/tasks
```

于是 CPU 就降到 20% 。

```
  PID       PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                       
16515       20   0    2336    728    544 R  20.0  0.0   1:27.23 baozou
```

除了这种需要指定 PID 来限制资源的方法，也可通过指定规则来执行，更显得方便，效果和上述一致。

```
sudo cgexec -g cpu:calm ./baozou
```

可以看看这个限制规则做了什么?

```
$ sudo cgget calm
calm:
cpu.shares: 1024
cpu.cfs_quota_us: 20000
cpu.stat: nr_periods 6943
    nr_throttled 6941
    throttled_time 563080015831
cpu.cfs_period_us: 100000
```

上述的例子中，我们手动创建了 `calm`， 其实也能通过命令来做到的

```
// 创建cgroup 文件目录
sudo cgcreate -g cpu:/calm -g memory:/calm

// 设置限制的参数
sudo cgset -r cpu.shares=200 calm

// 限制了内存
sudo cgset -r memory.limit_in_bytes=64k calm

// 可以删除
sudo cgdelete cpu/calm memory:/calm
```

---

参考链接：

* [Docker基础技术: Linux CGroup](http://coolshell.cn/articles/17049.html)
* [cgroup实践](http://www.jianshu.com/p/dc3140699e79)
