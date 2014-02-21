---
layout: post
title: "leveldb的RPCServer"
date: 2013-09-21 11:33
comments: true
categories: Server NetWork DataBase
---
leveldb作为一个高性能的存储引擎，并不提供Server以及网络的功能，但我坚信一定会有，因为这个世界有很多人会被其性能所吸引，总有人忍不住会写一个壳包装下，万能的github让我不用重复造轮子。[这个](https://github.com/srinikom/leveldb-server) ,还有[这个](https://github.com/quiver/leveldb-rpc/)

XML RPC 的代码很简单，但可以从中学到了不少东西的,主要使用的是 python 的[leveldb binding](http://zheng-ji.info//blog/2013/09/21/leveldbben-di-cun-chu-yin-qing-jing-zhi-de-gong-ju/) 

#### server.py

```python
'''XML-RPC server for leveldb
'''
import argparse
from SimpleXMLRPCServer import SimpleXMLRPCServer, SimpleXMLRPCRequestHandler

import leveldb

class RequestHandler(SimpleXMLRPCRequestHandler):
    rpc_paths = ('/RPC2',)

class LevelDBMethod(object):
    def __init__(self, datadir):
        self.db = leveldb.LevelDB(datadir)

    def put(self, key, value):
        return self.db.Put(key, value)

    def get(self, key):
        try:
        return self.db.Get(key)
        except KeyError, err:
        return ''

    def delete(self, key):
        return self.db.Delete(key)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--port', type=int, nargs='?', default=8000)
    parser.add_argument('--datadir', type=str, nargs='?', default='data')
    args = parser.parse_args()

    # Create server
    server = SimpleXMLRPCServer(("localhost", args.port),
                requestHandler=RequestHandler,
                allow_none=True)
    server.register_introspection_functions()
    server.register_instance(LevelDBMethod(args.datadir))
    server.serve_forever()

if __name__ == '__main__':
    main()
```

#### client.py

```python
'''XML-RPC client for leveldb
'''

import argparse
import cmd
from xmlrpclib import ServerProxy

class Cmd(cmd.Cmd):
    def __init__(self, host, port, *args, **kw):
        cmd.Cmd.__init__(self, *args, **kw)
        self.prompt = '%s:%d> ' % (host, port)
        self.server = ServerProxy('http://%s:%d' % (host, port))

    def do_put(self, arg):
        args = arg.split()
        if len(args) != 2:
            print "(error) ERR wrong number of arguments for 'put' command"
            return
        key, value = args
        result = self.server.put(key, value)

    def do_get(self, arg):
        args = arg.split()
        if len(args) != 1:
            print "(error) ERR wrong number of arguments for 'get' command"
            return
        key, = args
        result = self.server.get(key)
        if result:
            print result

    def do_delete(self, arg):
        args = arg.split()
        if len(args) != 1:
            print "(error) ERR wrong number of arguments for 'delete' command"
            return
        key, = args
        result = self.server.delete(key)

    def do_EOF(self, arg):
        return True

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument('--host', type=str, nargs='?', default='localhost')
    parser.add_argument('--port', type=int, nargs='?', default=8000)
    args = parser.parse_args()

    cmd = Cmd(args.host, args.port)
    cmd.cmdloop()

if __name__ == '__main__':
    main()
```

#### Server代码:
+ SimpleXMLRpcServer拓展,自定义业务逻辑类 LevelDBMethod,以及RPC运行路径别名,注册业务代码

```python
 # Create server
server = SimpleXMLRPCServer(("localhost", args.port),
        requestHandler=RequestHandler,
        allow_none=True)
server.register_introspection_functions()
server.register_instance(LevelDBMethod(args.datadir))
server.serve_forever()
```

+ argsparse解析命令行参数 
+ 包装leveldb ,公开RPC函数调用


#### Client代码:
+ 调用xmprpclib 的ServerProxy，完成网络RPC调用

```python
self.server = ServerProxy('http://%s:%d' % (host, port))
```

+ cmd模块，自定义命令行提示工具,调用cmdloop() 实现循环调用,继承cmd.Cmd类时，函数名为do_动作名
+ argsparse解析命令行参数 

```python
parser = argparse.ArgumentParser()
parser.add_argument('--host', type=str, nargs='?', default='localhost')
parser.add_argument('--port', type=int, nargs='?', default=8000)
args = parser.parse_args()
```

