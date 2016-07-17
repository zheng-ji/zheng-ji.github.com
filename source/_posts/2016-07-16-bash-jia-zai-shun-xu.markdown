---
layout: post
title: "环境变量的那些事"
date: 2016-07-16 11:35
comments: true
categories: System
description: ssh
---

* [四种模式下的环境变量加载](#第一节)
* [跨机器SSH 传递环境变量](#第二节)

<h3 id="第一节">四种模式下的环境变量加载</h3>

名词解析

1. login shell: 指用户以非图形化界面 ssh登陆到机器上时获得的第一个 shell。 
2. interactive: 交互式，有输入提示符，它的标准输入输出和错误输出都会显示在控制台上。

* interactive + login shell

比如登陆机器后的第一个 shell 就是这种场景。它首先加载 /etc/profile，然后再依次去加载下列三个配置文件之一，一旦找到其中一个便不再接着寻找

```
~/.bash_profile
~/.bash_login
~/.profile
```

设计如此多的配置是为了兼容 bourne shell 和 C shell，尽量杜绝使用 .bash_login，如果已创建，需要创建 .bash_profile 覆盖

* noninteractive + login shell

bash -l script.sh 就是这种场景。`-l` 参数是将shell作为一个login shell启动，配置文件的加载与第一种完全一样。

* interactive + non-login shell

在一个已有shell中运行bash，此时会打开一个交互式的shell，因为不再需要登陆，所以不是login shell。启动 shell 时会去查找并加载

```
/etc/bash.bashrc
~/.bashrc 
```

* non-interactive + non-login shell

比如执行脚本 bash script.sh 或者 ssh user@remote command。这两种都是创建一个shell，执行完脚本之后便退出，不再需要与用户交互。它会去寻找环境变量BASH_ENV，将变量的值作为文件名进行查找，如果找到便加载它。

从网上看到一个清晰的图

```
+----------------+--------+-----------+---------------+
|                | login  |interactive|non-interactive|
|                |        |non-login  |non-login      |
+----------------+--------+-----------+---------------+
|/etc/profile    |   A    |           |               |
+----------------+--------+-----------+---------------+
|/etc/bash.bashrc|        |    A      |               |
+----------------+--------+-----------+---------------+
|~/.bashrc       |        |    B      |               |
+----------------+--------+-----------+---------------+
|~/.bash_profile |   B1   |           |               |
+----------------+--------+-----------+---------------+
|~/.bash_login   |   B2   |           |               |
+----------------+--------+-----------+---------------+
|~/.profile      |   B3   |           |               |
+----------------+--------+-----------+---------------+
|BASH_ENV        |        |           |       A       |
+----------------+--------+-----------+---------------+
```
----

<h3 id="第二节">跨机器传递环境变量</h3>

假设要传递的变量叫做 $VARNAME

客户端机器的 `/etc/ssh_config` 添加 

```
SendEnv VARNAME
```

服务端机器的 `/etc/sshd_config` 添加

``` 
AcceptEnv VARNAME
```

客户端机器的 $VARNAME 就可以通过 ssh 传递到服务端机器，继续使用.

---

### 参考

[ssh连接远程主机执行脚本的环境变量问题](http://feihu.me/blog/2014/env-problem-when-ssh-executing-command-on-remote/)
