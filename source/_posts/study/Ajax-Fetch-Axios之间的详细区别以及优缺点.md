---
title: Ajax Fetch Axios之间的详细区别以及优缺点
date: 2018-07-18 11:26:47
tags: "Ajax"
---

将jQuery的ajax、axios和fetch做个简单的比较，所谓仁者见仁智者见智，最终使用哪个还是自行斟酌

# jQuery ajax

```js
$.ajax({
   type: 'POST',
   url: url,
   data: data,
   dataType: dataType,
   success: function () {},
   error: function () {}
})
```

> 优缺点：

* 本身是针对mvc模式的编程，不符合现在mvvm的浪潮

* 基于原生的XHR开发，XHR本身的架构不清晰，已经有了fetch的替代方案

* JQuery整个项目太大，单纯使用ajax却要引入整个JQuery非常的不合理（采取个性化打包的方案又不能享受CDN服务）

尽管JQuery对我们前端的开发工作曾有着（现在也仍然有着）深远的影响，但是我们可以看到随着VUE，REACT新一代框架的兴起，以及ES规范的完善，更多API的更新，JQuery这种大而全的JS库，未来的路会越走越窄。

# axios

[axios中文文档](https://www.kancloud.cn/yunye/axios/234845)

```js
axios({
    method: 'POST',
    url: ''
    data:{
        name:'zhang'
    }
}).then(function (response){
    console.log(response)
}).then(function (error){
    console.log(error)
})
```

<!-- more -->
Vue2.0之后，尤雨溪推荐大家用axios替换JQuery ajax，想必让Axios进入了很多人的目光中。Axios本质上也是对原生XHR的封装，只不过它是Promise的实现版本，符合最新的ES规范，从它的官网上可以看到它有以下几条特性：

> 优缺点：

* 从 node.js 创建 http 请求
* 支持 Promise API
* 客户端支持防止CSRF
* **提供了一些并发请求的接口**（重要，方便了很多的操作

这个支持防止CSRF其实挺好玩的，是怎么做到的呢，就是让你的每个请求都带一个从cookie中拿到的key, 根据浏览器同源策略，假冒的网站是拿不到你cookie中得key的，这样，后台就可以轻松辨别出这个请求是否是用户在假冒网站上的误导输入，从而采取正确的策略。

Axios既提供了并发的封装，也没有下文会提到的fetch的各种问题，而且体积也较小，当之无愧现在最应该选用的请求的方式。

# fetch

fetch号称是AJAX的替代品，它的好处在[《传统 Ajax 已死，Fetch 永生》](https://github.com/camsong/blog/issues/2)中提到有以下几点：

符合关注分离，没有将输入、输出和用事件来跟踪的状态混杂在一个对象里
更好更方便的写法，诸如：

```js
try{
  let response = await fetch(url);
  let data = response.json();
  console.log(data);
}catch (e){
    console.log("Oops, error", e);
}
```

> 优缺点：

* 符合关注分离，没有将输入、输出和用事件来跟踪的状态混杂在一个对象里
* 更好更方便的写法
* 更加底层，提供的API丰富（request, response）
* 脱离了XHR，是ES规范里新的实现方式

1）fetchtch只对网络请求报错，对400，500都当做成功的请求，需要封装去处理
2）fetch默认不会带cookie，需要添加配置项
3）fetch不支持abort，不支持超时控制，使用setTimeout及Promise.reject的实现的超时控制并不能阻止请求过程继续在后台运行，造成了量的浪费
4）fetch没有办法原生监测请求的进度，而XHR可以

PS: fetch的具体问题大家可以参考：[《fetch没有你想象的那么美》](http://undefinedblog.com/window-fetch-is-not-as-good-as-you-imagined/?utm_source=caibaojian.com)[《fetch使用的常见问题及解决方法》](https://www.cnblogs.com/huilixieqi/p/6494380.html)

看到这里，你心里一定有个疑问，这鬼东西就是个半拉子工程嘛，我还是回去用Jquery或者Axios算了——其实我就是这么打算的。但是，必须要提出的是，我发现fetch在前端的应用上有一项`xhr`怎么也比不上的能力：**跨域的处理**。

我们都知道因为同源策略的问题，浏览器的请求是可能随便跨域的——一定要有跨域头或者借助JSONP，但是，**fetch中可以设置mode为"no-cors"（不跨域）**，如下所示：

```js
fetch('/users.json', {
    method: 'post',
    mode: 'no-cors',
    data: {}
}).then(function() { /* handle response */ });
```

# 为什么要用axios

> axios 是一个基于Promise 用于浏览器和 nodejs 的 HTTP 客户端，它本身具有以下特征：

* 从浏览器中创建 XMLHttpRequest
* 从 node.js 发出 http 请求
* 支持 Promise API
* 拦截请求和响应
* 转换请求和响应数据
* 取消请求
* 自动转换JSON数据
* 客户端支持防止CSRF/XSRF