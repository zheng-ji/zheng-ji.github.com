---
layout: post
title: "记录使用 Flask 的点滴"
date: 2015-10-06 13:21
comments: true
categories: Programe
description: flask 经验
---

喜欢 Flask 经典的 RestFul 设计风格，以及它与 Gevent 的优雅结合，可以帮助我们轻松构建异步非阻塞应用，烂笔头记下一些较好的实践。

* [消息反馈](#第一节)
* [Flask 上下文](#第二节)
* [注册 JinJia 模板过滤器](#第三节)
* [itsdangerous 生成过期时间 Json 签名](#第四节)
* [一种较好的项目组织方式](#第五节)
* [BluePrint 的好](#第六节)
* [Json 返回](#第七节)
* [自定义出错页面](#第八节)
* [WTF 跨站脚本防御](#第九节)


<h3 id="第一节">消息反馈</h3>

Flask 提供了一个非常简单的方法来使用闪现系统向用户反馈信息。
闪现系统使得在一个请求结束的时候记录一个信息，然后在且仅仅在下一个请求中访问这个数据。这通常配合一个布局模板实现

[文档链接](http://docs.jinkan.org/docs/flask/patterns/flashing.html)

```
# 视图函数需要调用
flash('your response message for user')

# 前端页面调用
for message in get_flashed_messages()
就可以输出反馈信息
```

<h3 id="第二节">Flask 上下文</h3>

有两种上下文：程序上下文，请求上下文

* current_app： 程序级别上下文，当前激活程序的实例。
* g: 请求级别的上下文
* request: 是请求级别的上下文，封装了客户端发出的 Http 请求中的内容
* session: 用户会话，用于存储请求之间需要记住的键值对

<h3 id="第三节">注册 JinJia 模板过滤</h3>

```
def reverse_filter(s):
    return s[::-1]

app.jinja_env.filters['reverse'] = reverse_filter
```

<h3 id="第四节">itsdangerous 生成过期时间 Json 签名</h3>

```
serialaizer = Serializer(SECRET_KEY, expires_in=EXPIRES)
info = {'id':'xxx'}
session = serialaizer.dumps(info)

# 判断 session 时间
info = None
try:
    info = serialaizer.loads(session)
except Exception:
    return jsonify(ERR_SESS_TIMEOUT)
```

用途：生成一个有时间限制的签名，用于API 访问的验证码，如果过期，提醒用户重新登录

<h3 id="第五节">一种较好的项目组织方式</h3>

```
▾ app/
    ▾ controlers/
        ▾ admin/                    #管理后台的蓝图控制器
            ▾ forms/                #表单限制文件
                __init__.py　　　　　
                xx.py　　　         # blueprint 文件
            __init__.py
            administrator.py
        ▾ api/                      # API 控制器
            __init__.py
        ▾ site/                     # 站点控制器
            __init__.py             # blueprint 文件
            xx.py
        __init__.py
        error.py
    ▸ models/         # SQLAlchemy 的各类模型
    ▸ static/         # 需要的静态资源，css, js, imgs
    ▾ templates/　　　# 前端页面，跟 controller 分类一一对应
        ▸ admin/
        ▸ error/
        ▸ site/
    ▸ utilities/　　　　#  功能函数
      __init__.py       #　初始化app需要的各种插件，比如 redis, db, 注册蓝图
run.py                  #　相当于 main 函数,创建 app, 执行app.run() 函数
settings.py             #　配置文件
```

<h3 id="第六节">BluePrint 的好 </h3>

每个 Blueprint 就像独立的 Application, 可以管理自己的模板, 路由, 反向路由url_for, 甚至是静态文件，最后统一挂载到 Application 下。从头到尾都是 RestFul。

在创建 app (app/__init__.py) 的时候调用如下:

```
from app.controllers.site.console import console 
app.register_blueprint(console, url_prefix='/console')
```

视图文件 (app/controllers/site/console.py):

```
console = BLueprint('console', __name__)
```

<h3 id="第七节">Json 返回</h3>

```
from flask import jsonify
return jsonify({'code': 0})
```

<h3 id="第八节">自定义出错页面</h3>

```
@app.errorhandler(404)
def not_found(error):
    return render_template('404.html'), 404

@app.errorhandler(500)
def crash(error):
    return render_template('5xx.html'), 500
```

<h3 id="第九节">WTF 跨站脚本防御</h3>

Flask-WTF 能保护表单免受跨站请求伪造的攻击,恶意网站把请求发送到被攻击者已经登录的其他玩战会引发 CSRF 攻击

* app config 文件中，开启 CSRF 开关并配置密钥

```
CSRF_ENABLED = True
SECRET_KEY = 'you-will-never-guess'
```

* 表单的定义

```
from flask.ext.wtf import Form, TextField, BooleanField
from flask.ext.wtf import Required
class LoginForm(Form):
    openid = TextField('openid', validators = [Required()])
    remember_me = BooleanField('remember_me', default = False)
```

* 页面渲染

```
<form action="" method="post" name="login">
    form.hidden_tag()
    <p> Please enter your OpenID:form.openid(size=80)<br></p>
    <p>form.remember_me </p>
    <p><input type="submit" value="Sign In"></p>
</form>
```

* 控制器函数

```
@app.route('/login', methods = ['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        flash('Login requested for OpenID="' + form.openid.data))
        return redirect('/index')
    return render_template('login.html', title = 'Sign In',form = form)
```

我们在配置中开启了CSRF(跨站伪造请求)功能，模板参数 form.hidden_tag() 会被替换成一个具有防止CSRF功能的隐藏表单字段。 在开启了CSRF功能后，所有模板的表单中都需要添加这个模板参数
