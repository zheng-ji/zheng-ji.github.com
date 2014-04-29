---
layout: post
title: "shell技巧小结"
date: 2013-09-28 22:52
comments: true
categories: Programe
---
最近在工作上需要使用shell处理一些有趣的事情，觉得很好玩，希望这文章能一直后向积累,更新 ：） 分享快乐

* 使用shell获取进程，并删除

```
#取得当前进程号
current_pid=$$
#获得特定进程号并重定向到一个临时文件中
ps -aux|grep "/usr/sbin/httpd" | grep -v "grep"| awk '{print $2}' >/tmp/${current_pid}.txt
#commands start
for pid in `cat /tmp/${current_pid}.txt`
do
{
    echo "kill -9 $pid"
    kill -9 $pid
}
done 
rm -f /tmp/${current_pid}.txt 
```

* shell的函数处理

```
func()
{
    echo $a $b $c
    echo $1 $2 $3
}
a="aaa"
b="bbb"
c="ccc"
func $a e f
```

函数传参是通过 funcname $para1 $para2 ...,函数内部是使用$1,$2,$3
结果为：

```
aaa bbb ccc
aaa e f 
```

* 信号捕捉
trap可以使你在脚本中捕捉信号，命令形式为trap name signal(s)
其中,name是捕捉到信号以后所采取的一系列操作。实际中，name一般是一个专门用来处理所捕捉信号的函数。name需要用双引号（“”）引起来。signal是待捕捉的信号。
最常见的行动包括： 清除临时文件 忽略该信号(如trap "" 23) 询问用户是否终止该脚本进程

```
trap 'exitprocess' 2　　　　#捕捉到信号２之后执行exitprocess function
LOOP=0
function exitprocess()
{
    echo "You Just hit <CTRL-C>, at number $LOOP"
    echo "I will now exit"
    exit 1
}
while : # 循环直到捕捉到信号（注意中间的空格） 
do
    LOOP=$LOOP+1
    echo $LOOP
    sleep 1
done 
```

 给一个文本，获取其中2000 ~ 3000 行怎么破？

```
cat data.txt | tail -n +2000 | head -n 1000 > result.txt
```

查找文件是否存在
```
find /data -name hello.cpp 
```

