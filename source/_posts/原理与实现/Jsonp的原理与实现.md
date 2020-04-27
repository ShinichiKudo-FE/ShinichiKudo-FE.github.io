---
title: Jsonp的原理与实现
date: 2018-05-03 10:36:51
tags: "jsonp"
categories: "原理与实现"
---
# 概述

Jsonp是一种跨域通信的手段，它的原理实现很简单：

* 1.首先利用script标签的src属性来实现跨域
* 2.通过将前端方法作为参数传给服务器端，由服务器端注入参数之后再返回，实现服务器端向客户端通信
* 3.由于使用script的src属性，所以只支持get方法

## 实现流程

1.设定一个script标签

```js
<script src="http://jsonp.js?callback=xxx"></script>
```

2.callback定义了一个函数名，而远程服务端通过调用指定的函数并传入参数实现参数传递，将fn(response)传递回客户端

```js
$callback = !empty($_GET['callback']) ? $_GET['callback']: 'callback';
echo $callback.'(.json_encode($data))';
```

3.客户端接收到返回的js脚本，开始解析和执行fn(response)
<!-- more -->

## jsonp的简单实现

一个简单的jsonp实现，其实就是拼接url字符串，动态的添加到script元素的头部

```js
function jsonp(req){
    var script = document.createElement('script');
    var url = req.url + '?callback=' + req.callback.name;
    script.src = url;
    document.getElementsByTagName('head')[0].appendChild('script');
}
```

前端js示例

```js
function hello(res){
    alert('hello' + res.data);
}
jsonp({
    url:'',
    callback: hello
})
```

服务器端代码

```js
var http = requrie('http');
var urllib = require('url');
var port = 8080;
var data = {'data':'world'};

http.createServer(function(req,res){
    var params = urllib.parse(req.url,true);
    if(params.query.callback){
        console.log(params.query.callback);
        //jsonp
        var str = params.query.callback + '(' + JSON.stringify(data) + ')';
        res.end(str);
    } else {
        res.end();
    }
}).listen(port,function(){
    console.log('jsonp server is on');
});

```

然而，这个实现虽然简单，但有一些不足的地方：

1.我们传递的回调必须是一个全局方法，我们都知道要尽量减少全局的方法。
2.需要加入一些参数校验，确保接口可以正常执行。

## 可靠的jsonp实现

jsonp部分源码

```js
(function (global) {
    var id = 0,
        container = document.getElementsByTagName("head")[0];

    function jsonp(options) {
        if(!options || !options.url) return;

        var scriptNode = document.createElement("script"),
            data = options.data || {},
            url = options.url,
            callback = options.callback,
            fnName = "jsonp" + id++;

        // 添加回调函数
        data["callback"] = fnName;

        // 拼接url
        var params = [];
        for (var key in data) {
            params.push(encodeURIComponent(key) + "=" + encodeURIComponent(data[key]));
        }
        url = url.indexOf("?") > 0 ? (url + "&") : (url + "?");
        url += params.join("&");
        scriptNode.src = url;

        // 传递的是一个匿名的回调函数，要执行的话，暴露为一个全局方法
        global[fnName] = function (ret) {
            callback && callback(ret);
            container.removeChild(scriptNode);
            delete global[fnName];
        }

        // 出错处理
        scriptNode.onerror = function () {
            callback && callback({error:"error"});
            container.removeChild(scriptNode);
            global[fnName] && delete global[fnName];
        }

        scriptNode.type = "text/javascript";
        container.appendChild(scriptNode)
    }

    global.jsonp = jsonp;

})(this);
```

使用示例

```js
jsonp({
    url: 'www.example.com',
    data: {id :1},
    callback: function(ret){
        console.log(ret)
    }
})
```

## JSONP的优缺点

### 优点

* 它不像XMLHttpRequest对象实现的Ajax请求那样受到同源策略的限制，JSONP可以跨越同源策略；
* 它的兼容性更好，在更加古老的浏览器中都可以运行，不需要XMLHttpRequest或ActiveX的支持
* 在请求完毕后可以通过调用callback的方式回传结果。将回调方法的权限给了调用方。这个就相当于将controller层和view层终于分 开了。我提供的jsonp服务只提供纯服务的数据，至于提供服务以 后的页面渲染和后续view操作都由调用者来自己定义就好了。如果有两个页面需要渲染同一份数据，你们只需要有不同的渲染逻辑就可以了，逻辑都可以使用同 一个jsonp服务。

### 缺点

* 它只支持GET请求而不支持POST等其它类型的HTTP请求
* 它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行JavaScript调用的问题。
* jsonp在调用失败的时候不会返回各种HTTP状态码。
* 缺点是安全性。万一假如提供jsonp的服务存在页面注入漏洞，即它返回的javascript的内容被人控制的。那么结果是什么？所有调用这个 jsonp的网站都会存在漏洞。于是无法把危险控制在一个域名下…所以在使用jsonp的时候必须要保证使用的jsonp服务必须是安全可信的

### 安全防范

* 防止callback参数意外截断js代码,特殊字符单引号双引号,换行符存在风险.
* 防止callback参数恶意添加script标签,造成xss漏洞
* 防止跨域请求滥用,阻止非法站点恶意调用
