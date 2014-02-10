---
layout: post
title: "微信公众号开发"
date: 2014-01-07 18:47
comments: true
categories: Server Product
keywords: 微信公众号开发
description: 微信公众号开发 
---
### 微信公众账号开发

话说2013年的最后几分钟和2014的最开始几分钟我都是献给这货的.BTW,纯属个人折腾
> 人生苦短,我爱python

至于做的具体东西就在此略过了,以下是截图.主要是分享开发过程中的经验,我的代码是部署在自己的VPS上面

{% img /images/2013/12/nba.jpg %}

现在注册一个微信的服务号的成本很高了,一次人工审批不过的话就需要300元, 我用的是微信公众平台测试号.只要填写手机号和验证码。其限制条件是,只能有20个订阅粉丝.没关系,我要的是调通整个通讯以及业务化定制.

#####注册
会给你一个APPID,APPSeceret,这个步骤需要你填写自己Server的URL,以及填写TOKEN,这些是未来完成通讯的许可证明

#####微信认证
Server 端应该对每个请求进行验证，确认是来自微信服务器的请求才做出相应

加密/校验流程如下：

* 将token、timestamp、nonce三个参数进行字典序排序
* 将三个参数字符串拼接成一个字符串进行sha1加密
* 开发者获得加密后的字符串可与signature对比，标识该请求来源于微信

```python
class BaseHandler(tornado.web.RequestHandler):
    #基类,实现基本认证，子类只需要继承就可以完成微信认证
    def prepare(self):
        timestamp = self.get_argument("timestamp", None)
        nonce = self.get_argument("nonce", None)
        signature = self.get_argument("signature", None)
        print 'request:', timestamp, nonce, signature
        print self.request.method, timestamp, nonce, signature
        if validate(timestamp, nonce, signature) is False:
            print "=========validate failed ====="
            self.finish()

def validate(timestamp, nonce, signature):
    print 'validate:', timestamp, nonce, signature
    if timestamp is None or nonce is None or signature is None:
        return False
    tmp = [TOKEN, timestamp, nonce]
    tmp = sorted(tmparr)
    tmpStr = ''.join(tmp)
    hashStr = sha1(tmpStr).hexdigest()
    return (hashStr == signature)
```
验证成功之后，才能进入业务逻辑,当我们在微信公众号填写自己服务的URL的时候，weixin会向该URL发起Get请求.做为首次验证,根据文档，需要将接收到的字符串原文返回
```
class serverHandler(BaseHandler):
    def get(self):
        echostr = self.get_argument('echostr', None)
        self.write(echostr)
``` 


#####. 菜单订制
 为了支持微信通信过程中不允许使用"/u" 字符编码，这里用到了tornado的render_string,一开始我是直接生成一个json.dump(menu)然后post过去,但一直存在编码问题， 使用render_string就解决了。感谢@ihao提醒
```
class UIHandler(tornado.web.RequestHandler):
    def get(self):
        msg = self.render_string('menu.json')
        createMenu(menu)
```


#####. 响应用户按键输入
```
xml = uni(self.request.body)        #获取微信服务器发送过来的xml,转化为unicode
dic = xml2dict(xml)                 #将xml解析成dict
if dic['MsgType'].lower() == 'event':
    event = dic['Event'].lower()
    if event == 'subscribe':
        ctx = do_subscribe(dic)     #订阅逻辑
        msg = self.render_string('text_resp.xml', dic=ctx)
        self.write(msg)
    elif event == 'click':
        ctx = do_click(dic)         #点击逻辑
        msg = self.render_string('text_resp.xml', dic=ctx)
        self.write(msg)
```
self.write(msg)是将要发给用户的信息返回给微信服务器，之后微信服务器会将信息发送给用户


#### 相关链接
- [开发者文档](http://mp.weixin.qq.com/wiki/index.php)
- [接口调试](http://mp.weixin.qq.com/debug/)


#### 开发中用到的库
- tornado
- virtualenv
- pyquery(做爬虫)
- pymongo
- lxml
- urllib2


