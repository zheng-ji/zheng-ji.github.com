---
layout: post
title: "Gtalk机器人"
date: 2012-12-29 13:32
comments: true
categories: Programe
---
#### 缘起：
最近工作需要即时发送报警信息到gtalk，工作中使用gtalk作为通讯工具，所以催生了此项目开发 

#### 所用的工具:
python-xmpp库，tornado(httpserver 框架),其实也可以自己写

#### 什么是xmpp
XMPP（可扩展消息处理现场协议）是基于可扩展标记语言（XML）的协议，它用于即时消息（IM）以及在线现场探测。它在促进服务器之间的准即时操作。这个协议可能最终允许因特网用户向因特网上的其他任何人发送即时消息，即使其操作系统和浏览器不同XMPP的前身是Jabber，一个开源形式组织产生的网络即时通信协议

#### 关键代码
```
#robot 开始发送消息
def chat(msg):
    jid = xmpp.JID(user)
    connection = xmpp.Client(server,debug=[])
    connection.connect()
    result = connection.auth(jid.getNode(), password,"LFY-client")
        connection.sendInitPresence()
 
    fp = open("contactlist","r")
        for line in fp:
                to = line[0:-1]
                        message = xmpp.Message(to, msg)
        message.setAttr('type', 'chat')
                connection.send(message)
    fp.close()
 
if __name__ == '__main__':
    msg = "please ignore me ! i am just test"
        chat(msg)
```
 这样就可以自动发信息给联系人了，contactlist里面为通讯录名单，在项目中把上述代码实现为http接口，让监控端调用。由于只是简单的应用，我甚至用文本替代数据库，不过这也可以试一试面向对象的编程思想.

代码[链接](https://github.com/zheng-ji/gtalk-robot)
Happy Hacking!
