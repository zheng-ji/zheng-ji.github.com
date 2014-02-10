---
layout: post
title: "Jsonp解决跨域请求"
date: 2013-09-15 21:50
comments: true
categories: NetWork
---

昨天搞了一整天js,遇到了各种我从没遇到的问题,真心觉得前端工程师是很厉害的。想不出为什么会有人说所谓的前端“无技术含量”的源论.前端要考虑的东西实在是多，也有其架构设计。特别是js. 我用的是phonegap + jqtouch做移动App


场景是：
客户端使用Ajax技术向服务器发送请求，然后拉取数据

Server端代码是php写的,

```php
$arr = array(
        'user'=>'123',
        'friends' => 234
        )
echo json_encode($arr);
```

js代码,使用jquery发起get 请求,文件路径E:/mytest/test.html

```javascript
$(function(){
    url = "http://127.0.0.1/test.php"
    $.get(url,funtion(result) {
        alert(result);
    })
);
```

看过jquery文档之后，我以为这样会返回我要的数据，真是图样图森破，chrome的控制台返回：不是同一个源，不允许跨域发请求。 

#####什么是跨域请求
简单来说：a.com 域名下的js无法操作b.com或是c.a.com域名下的对象,上述场景中，http://127.0.0.1/test.php 和 E：/mytest/test.html本身就不属于同一个域下，因此不能访问。浏览器本身有同源策略，当初设计是出于安全考虑，百度给出了解释是：
>　同源策略，它是由Netscape提出的一个著名的安全策略。现在所有支持JavaScript 的浏览器都会使用这个策略。所谓同源是指，域名，协议，端口相同。当一个浏览器的两个tab页中分别打开来 百度和谷歌的页面当一个百度浏览器执行一个脚本的时候会检查这个脚本是属于哪个页面的，即检查是否同源，只有和百度同源的脚本才会被执行。

我最后是通过查阅资料,采用jsonp的方式解决

#####什么是jsonp
>　JSONP是JSON with Padding的略称。它是一个非官方的协议，它允许在服务器端集成Script tags返回至客户端，通过javascript callback的形式实现跨域访问（这仅仅是JSONP简单的实现形式） - 百度

因此最后的Server端代码是php写的

```python
$arr = array(
        'user'=>'123',
        'friends' => 234
        )
$callback = $_GET['callback']; //新加
echo $callback.json_encode($arr);
```

js代码,请求的时候，加入一个参数callback,通过javascripte callback形式调用服务器的scrpt tags的返回值

```javascript
$(function(){
    url = "http://127.0.0.1/test.php?callback=?"
    $.get(url,funtion(result) {
        alert(result);
    }));
```

调用成功
这个东西纠结我大半天。但是有收获的 ：）





