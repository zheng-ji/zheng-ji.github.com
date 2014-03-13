---
layout: post
title: " Expect 与系统自动交互"
date: 2014-03-13 23:05
comments: true
categories: Programe
keywords: expect语言
description: expect语言
---

在`linux`下 做一些操作的时候，比如使用 *scp*, *telnet* 等命令，通常系统会让你输入密码，你只好乖乖地停下来输入。倘若你希望批量执行一个脚本中得所有命令，系统执行到某句命令的时候停下来让你输入密码，就比较囧了。于是你感慨，如果是一气呵成多省事啊。

这个时候，你可以使用 *expect* 语言帮我们解决问题。

###官方定义
> Expect是一个用来实现自动交互功能的软件套件 (Expect [is a] software suite for automating interactive tools)

expect 是 [Don Libes](http://en.wikipedia.org/wiki/Don_Libes) 编写的免费编程工具语言, 由expect-send对组成：

expect等待输出中输出特定的字符，通常是一个提示符，然后发送特定的响应。

-------------------------------------------------

先来个 helloworld, 注意需要在exp脚本前面加上 `#!/usr/bin/expect`

```sh
#!/usr/bin/expect
send_user "hello world"
```

这里的 send_user 类似 echo 

### 常见用法

```
set i 0    
set user [lindex $argv 0]
set timeout 300
set i 0
while { $i < 10 } {
    puts $i;
    incr i;
}
```

以上分别是:

+ 将0赋值给变量i;
+ 获取脚本的第一个参数给变量user;
+ 设置超时时间是300秒
+ 循环使用，不多说

-------------------------

在 expect脚本中, 最亮点的是 spawn 会启动一个进程，由 expect 和 send 组合负责跟该进程交互, in short 

+ expect捕获进程需要什么信息
+ send将这个信息发给进程
+ expect eof与spawn对应，表示捕获终端输出信息的终止，即进程已结束
+ expect都是使用 {}，且{, }使用时，前后需要留空格。 只要其中一项符合，就会执行该项，类似switch

看完以下代码相信你就明白其魅力了 ：）

```sh
#!/usr/bin/expect
  
set timeout 10
set host [lindex $argv 0]
set username [lindex $argv 1]
set password [lindex $argv 2]
 
spawn ssh $username@$host
expect {
    "(yes/no)?" {
        send "yes\n"
        expect "*assword:" { send "$password\n" }
    }
    "*assword:" {
        send "$password\n"
    }
}

expect "$"
expect eof
```

使用时, 执行如下脚本命令.  其中`expect` 捕获到 "*assword:", 发送 `$password` 变量,  捕获到`$`符号，表示登陆成功，结束 spawn 进程

```
./autosh.exp xxx(ip地址) xxx(用户名) xxx(密码)
```

-----------------------------
感觉这样的脚本在自动化运维可以有很多运用之处, 至少可以让你可以在执行需要交互命令的时候可以跑去喝茶了。

Happy Hacking :-)


