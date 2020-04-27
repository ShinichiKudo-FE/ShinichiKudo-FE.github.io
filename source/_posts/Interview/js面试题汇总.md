---
title: js面试题汇总
date: 2018-03-12 22:31:11
tags: "Interview"
categories: "Interview"
---

##  闭包是什么？闭包的用处。

 闭包是内部函数对外部函数作用域的引用。而在函数外部引用函数内部变量，可以通过闭包实现。闭包的用处往往是在模块封装的时候。可以将模块内部公有部分暴露出来。(闭包是基础性的问题，这里只是简要的阐述了一下它的概念) [闭包详解(阮一峰)](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

## 原型以及原型链

js与常用的编程语言不一样，他是靠原型继承的，并不是类。使用造零件的方式进行类比，通过类继承的方式就是通过一个模型盒铸件，而原型的方式，则是从另一个零件克隆一份，在这基础上进行修改。而那份被克隆的零件就是原型 [原型及原型链详解](https://segmentfault.com/a/1190000000662547)。原型链就是它们之间产生的联系，往往可以通过画图来阐述：
![protype](https://camo.githubusercontent.com/2e0aa05f5710b00399841b284e41de3f5c866a0b/687474703a2f2f6f6966737635696a692e626b742e636c6f7564646e2e636f6d2f70726f746f747970652e706e67)

## 垃圾回收机制和内存泄漏

js引擎有自己的一套垃圾回收的机制，主要的回收机制有两种标记清除和循环引用。标记清除，即是在局部变量在使用过程中打上标记，一旦离开了执行环境，标记将会被清除，这是垃圾回收器将会知道，该变量已经不使用，可回收。而循环引用是在较早版本的IE浏览器中被使用，即初始化的变量引用数为0，一旦被赋值给另一个变量，引用就加1；而另一个变量一旦被赋予其他值，则该值的引用数减一。垃圾回收器会去判断该变量的引用数是否为0，为0，则会被回收掉。这个机制会存在循环引用的问题，即两个变量相互引用。[垃圾回收](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)

内存泄漏的问题。可以罗列为：

<!-- more -->

无意识的变量使用

```javascript
function func(){
   i = 2;
}
```

这种方式在严格模式下是不生效的。
2. 闭包导致的问题

其实，闭包导致的内存泄漏主要是在IE引用dom元素的时候。IE中的部分dom元素并非原生的，而是COM。这时使用，往往会导致循环引用，而导致内存泄漏。还有就是无意识的使用闭包导致的问题。1. 在函数内部添加时间时，会造成dom元素的引用，而导致内存泄漏；2. 在删除dom时，在之前不小心添加了事件等情况[内存泄漏](http://www.cnblogs.com/sprying/archive/2013/05/31/3109517.html)

## 柯里化

js的柯里化就是将函数的部分参数分成多次进行调用。例子：[柯里化](https://mp.weixin.qq.com/s?__biz=MzAwNjI5MTYyMw==&mid=2651494750&idx=1&sn=ccc6108723a246099acec6b523eefa54&chksm=80f19096b7861980c6ad788321c68b805e189ce59e828c6439597a1ea250a2762b5de53afe94&scene=38#wechat_redirect)

## 异步

面试之中主要会问你何为异步？其实，js是一门单线程的语言，正常的方式是整个程序按照顺序一步一步的执行下去，这就是所谓的同步模式，但是往往js中会有许多的任务耗时比较长(比如说ajax请求)等，如果按照同步的方式，往往会导致浏览器的无响应。这时，就需要通过异步的形式。异步模式：js的任务往往具备一个或多个回调函数，在执行的过程中，后一个任务无需前一个任务执行返回结果，而是继续执行，之后前一个任务可以调用回调函数，将结果返回回来。[异步](http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html)

## setTimeout和setInterval的区别，以及setTimeout的原理，setInterval会造成什么问题

首先，我们来应该了解一下setTimeout的原理。浏览器是一个事件为主体的事物。总是有一个队列在不停的循环。当使用setTimeout时，浏览器会创建一个定时器，然后将函数中的回调函数放入handle队列里面。然后浏览器会不停的循环整个handle，一旦handle中的内容满足条件，就会调用其中的回调函数。setTimeout的满足条件就是它后面的设置的时间差。

setTimeout和setInterval之间的区别主要是setTimeout是一个延迟函数，只执行一次，而setInterval会间隔一定的时间反复的执行。[setTimeout和setInterval的区别](http://www.cnblogs.com/tugenhua0707/p/4083475.html)

setInterval的原理，其实与之前的setTimeout的原理有点类似。只是在每个Event Loop之后，都会去检测是否满足条件，如果满足条件的话，就执行回调函数，如果不满足，放入下一轮的event loop中。主要的问题就是，如果一个setInterval中的回调函数不能够被执行，会引起系统阻塞的问题，之后的所有的定时器都会积累起来，之后就会直接执行，没有时间间隔。[setInterval机制](https://jeffjade.com/2016/01/10/2016-01-10-javaScript-setInterval/)

```javascript
function interval(func, wait){
    var interv = function(){
         func.call(null);
         setTimeout(interv, wait);
   };
   setTimeout(interv, wait);
}

interval(() => {
    console.log(1);
}, 1000);      //1 1 1
```

## postmessage和iframe怎么结合使用
postmessage是html5新出的一个API，可以解决多窗口、页面与新标签、窗口和iframe之间的消息通信的跨域问题。使用方法就是postmessage(data, origin)，其中data指的就是需要传递的数据，origin指的是具体的数据源地址(包括协议+域名+端口)。然后window对message事件进行监听。[postMessage解决跨域](http://www.cnblogs.com/dolphinX/p/3464056.html)

## js模板引擎

js模版引擎，其实就是预处理器，将一些字符串去匹配数据，然后将数据插入到固定的html模版之中。js的模版引擎主要是使用正则表达式去过滤一些字符串，然后将里面的每个语句片段都放到固定的数组之中，然后在对数组进行解析，之后拼成一个固定的js代码，将内容返回出来。著名的模版引擎[ejs](http://ejs.co/)。[js模版引擎原理解析](http://www.cnblogs.com/hustskyking/p/principle-of-javascript-template.html)

## WebSocket？长轮询和短轮询

webSocket是一种网络通信协议，基于TCP/IP的另一种通信协议(还有一种是http协议)。它与http的主要区别就是，http是单向通信的，客户端可以向服务器发送请求，但是，服务端如果有消息，无法向客户端推送。在没有websock之前，都是通过http定期发送请求的方式——轮询。轮询的效率是相对较低的。[websock协议](http://javascript.ruanyifeng.com/htmlapi/websocket.html)

之后的长轮询和短轮询呢？这个问题还得更实际的场景相结合起来。首先，我们在写电商网站的时候，商品的库存是实时变化的，这时，你要和服务器同步这个变化，才能够使得信息正确。那么，最简单的方式就是我们写一个ajax请求，不断的去给服务器请求，同步这个数据。而服务器每次接收到请求就返回一次信息。这就是所谓的短轮询，这种方式其实非常的消耗服务器的资源，试想服务器每次接收到请求就要去查询数据库，获取信息，而往往库存是没有这么快改变的。那么，或许长轮询可以更加方便地解决这个问题。我们可以写一个ajax请求，同时给这个请求设置超时机制，一旦超时之后，再次重新发送。而服务器每次接收到这个请求，只需要将之挂起，等检测到库存变化时，在响应这个请求，这样就有效的降低了轮询的次数。同样的问题，服务器的资源还是会有所损耗，毕竟一个请求相当于一个线程。对于大型的电商网站来说，这不是一种好的解决方式。[长轮询、短轮询、长链接、短链接](http://web.jobbole.com/85541/)

## cookie、LocalStorage 和 SessionStorage 的概念以及它们之间的区别

首先，cookie是我们比较熟悉的，是浏览器与服务器之间数据传递的方式，之前通常也会用来做一些数据的缓存，例如：用户的用户名等。但是cookie在做数据缓存的时候往往有缺点：

cookie数据最多只能缓存4kb，大于4kb的cookie会被浏览器默认弃用
cookie在做数据缓存时，往往会将数据发送给服务器，但是，有的时候这些数据并不需要发送给服务器
cookie在每个域中的数量是有限的，往往超过一定数量，那么之前的cookie将被抛弃
html5新特性中就带来了相应的客户端storage，就是localStorage和sessionStorage。localStorage是本地存储，数据不会发送给服务器，同时它的缓存大小大约在5M左右，数据是永久存储的，除非手动删除。sessionStorage则是会话存储，大部分的特性和localStorage一样，但是，sessionStorage中的内容会在浏览器或者页面关闭之后被清除掉。[cookie、LocalStorage和SessionStorage](http://harttle.land/2015/08/16/localstorage-sessionstorage-cookie.html)

## 浏览器事件代理的原理

其实，事件代理和事件委托是同一个概念。举个例子说明或许会更加形象。场景是：DOM结构为一个列表，而你需要在点击每个li标签时，在控制台输出它的内容。这时，你或许会在每个li上面添加点击事件，然后输出每个li的内容。但是，往往这样子做，会使得整个应用的性能降低，而且使得添加过程繁琐，不灵活。我们可以使用事件代理的方式，在整个列表的ul上面去添加点击事件，然后在内部使用target(真实点击元素)来使其输出它的内容。这样，我们就可以只写一个事件，而完成多个li之间的内容输出。事件代理的原理是使用了事件冒泡的特性。但是，有时候有些事件没有冒泡机制，如focus，blurs等，就不能使用事件代理，还有些类似于mouseover等事件，会导致实时的触发，也不宜使用事件代理[事件代理](http://le0zh.github.io/2016/06/04/event-delegate-in-javascript/)

## 前端安全 XSS，CSRF ？避免方法？

XSS，被称为跨站脚本攻击，主要分为三种反射型、存储型和DOM型。反射型主要是一些恶意链接的发送，存储型主要出现在评论等，可以插入到服务器数据库中，这种攻击比较持久；DOM型非常的少见。XSS主要是盗取用户的cookie等数据。CSRF，被称为跨站伪造请求攻击，主要与xss相结合[XSS和CSRF](http://www.freebuf.com/articles/web/39234.html)

## 前端性能优化的方法

## 同源和跨域

## 面向对象和继承

## object.create的实现原理

## object的深拷贝和浅拷贝

## 面向切面编程和函数式编程

[原文链接](https://github.com/laizimo/zimo-article/issues/15)
