---
title: Node入门
date: 2018-03-28 17:03:39
tags: "Node"
categories: "Node"
---
# 什么是Node.js

按照 Node.js官方网站主页 的说法:
> Node.js® is a JavaScript runtime built on Chrome's V8 JavaScript engine. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.
> 从上面这段话来看：
1.Node.js 不是 JavaScript 应用，不是语言（JavaScript 是语言），不是像 Rails(Ruby)、 Laravel(PHP) 或 Django(Python) 一样的框架，也不是像 Nginx 一样的 Web 服务器。Node.js 是 JavaScript 运行时环境

2.构建在 Chrome's V8 这个著名的 JavaScript 引擎之上，Chrome V8 引擎以 C/C++ 为主，相当于使用JavaScript 写法，转成 C/C++ 调用，大大的降低了学习成本

3.事件驱动（event-driven），非阻塞 I/O 模型（non-blocking I/O model），简单点讲就是每个函数都是异步的，最后由 Libuv 这个 C/C++ 编写的事件循环处理库来处理这些 I/O 操作，隐藏了非阻塞 I/O 的具体细节，简化并发编程模型，让你可以轻松的编写高性能的Web应用，所以它是轻量（lightweight）且高效（efficient）的

4.使用 npm 作为包管理器，目前 npm 是开源库里包管理最大的生态，功能强大，截止到2017年12月，模块数量超过 60 万+

大多数人都认为 Node.js 只能写网站后台或者前端工具，这其实是不全面的，Node.js的目标是让并发编程更简单，主要应用在以网络编程为主的 I/O 密集型应用。它是开源的，跨平台，并且高效（尤其是I/O处理），包括IBM、Microsoft、Yahoo、SAP、PayPal、沃尔玛及GoDaddy都是 Node.js 的用户。

<!-- more -->

## 基本原理

下面是一张 Node.js 早期的架构图，来自 Node.js 之父 Ryan Dahl 的演讲稿，在今天依然不过时，它简要的介绍了 Node.js 是基于 Chrome V8引擎构建的，由事件循环（Event Loop）分发 I/O 任务，最终工作线程（Work Thread）将任务丢到线程池（Thread Pool）里去执行，而事件循环只要等待执行结果就可以了。

![node1](https://i5ting.github.io/How-to-learn-node-correctly/media/14912707129964/14912763353044.png)

>核心概念:
1.Chrome V8 是 Google 发布的开源 JavaScript 引擎，采用 C/C++ 编写，在 Google 的 Chrome 浏览器中被使用。Chrome V8 引擎可以独立运行，也可以用来嵌入到 C/C++ 应用程序中执行。
2.Event Loop 事件循环（由 libuv 提供）
3.Thread Pool 线程池（由 libuv 提供）

梳理后可以这样理解：
1.Chrome V8 是 JavaScript 引擎
2.Node.js 内置 Chrome V8 引擎，所以它使用的 JavaScript 语法
3.JavaScript 语言的一大特点就是单线程，也就是说，同一个时间只能做一件事
4单线程就意味着，所有任务需要排队，前一个任务结束，才会执行后一个任务。如果前一个任务耗时很长，后一个任务就不得不一直等着。
5.如果排队是因为计算量大，CPU 忙不过来，倒也算了，但是很多时候 CPU 是闲着的，因为 I/O 很慢，不得不等着结果出来，再往下执行
6.CPU 完全可以不管 I/O 设备，挂起处于等待中的任务，先运行排在后面的任务
7.将等待中的 I/O 任务放到 Event Loop 里
8.由 Event Loop 将 I/O 任务放到线程池里
  只要有资源，就尽力执行

## 学习 Node.js 的三个境界

1.打日志 console.log
2.断点调试：断点调试：node debugger 或node inspector 或vscode
3.测试驱动开发（tdd | bdd）

## 安装Node.js环境

3m安装法

nvm（node version manager）【需要使用npm安装，替代品是yrm（支持yarn）】
nrm（node registry manager）【需要使用npm安装，替代品是yrm（支持yarn）】
npm（node packages manager）【内置，替代品是n或nvs（对win也支持）】

## Node.js应用场景

《Node.js in action》一书里说，Node.js 所针对的应用程序有一个专门的简称：**DIRT**。它表示数据密集型实时（data-intensive real-time）程序。因为 Node.js 自身在 I/O 上非常轻量，它善于将数据从一个管道混排或代理到另一个管道上，这能在处理大量请求时持有很多开放的连接，并且只占用一小部分内存。它的设计目标是保证响应能力，跟浏览器一样。

这话不假，但在今天来看，DIRT 还是范围小了。其实 DIRT 本质上说的 I/O 处理的都算，但随着大前端的发展，Node.js 已经不再只是 I/O 处理相关，而是更加的“Node”！

Node.js 使用场景主要分为4大类
![node2](https://i5ting.github.io/How-to-learn-node-correctly/media/14912707129964/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202017-05-17%2007.25.05.png)

1）跨平台：覆盖你能想到的面向用户的所有平台，传统的PC Web端，以及PC客户端 nw.js/electron 、移动端 cordova、HTML5、react-native、weex，硬件 ruff.io 等
2）Web应用开发：网站、Api、RPC服务等
3）前端：三大框架 React \ Vue \ Angular 辅助开发，以及工程化演进过程（使用Gulp /Webpack 构建 Web 开发工具）
4）工具：npm上各种工具模块，包括各种前端预编译、构建工具 Grunt / Gulp、脚手架，命令行工具，各种奇技淫巧等

| 分类        | 描述   |  相关模块  |
| --------   | -----:  | :----:  |
| 网站    | 类似于 cnodejs.org 这样传统的网站 |   Express / Koa    |
| Api        |   同时提供给移动端，PC，H5 等前端使用的 HTTP Api 接口   |   Restify / HApi   |
| Api代理       |    为前端提供的，主要对后端Api接口进行再处理，以便更多的适应前端开发    |  Express / Koa |
| IM即时聊天    | 实时应用，很多是基于 WebSocket协议的 |   Socket.io / sockjs     |
| 反向代理        |   提供类似于 nginx 反向代理功能，但对前端更友好  |   anyproxy / node-http-proxy / hiproxy   |
| 前端构建工具       |    辅助前端开发，尤其是各种预编译，构建相关的工具，能够极大的提高前端开发效率    |  Grunt / Gulp / Bower / Webpack / Fis3 / YKit  |
| 命令行工具    | 使用命令行是非常酷的方式，前端开发自定义了很多相关工具，无论是shell命令，node脚本，还是各种脚手架等，几乎每个公司\小组都会自己的命令行工具集 |   Cordova / Shell.js     |
| 操作系统       |  有实现，但估计不太会有人用  |   NodeOS   |
| 跨平台打包工具       |   使用 Web 开发技术开发PC客户端是目前最流行的方式，会有更多前端开发工具是采用这种方式的    |  PC端的electron、nw.js，比如钉钉PC客户端、微信小程序IDE、微信客户端，移动的Cordova，即老的Phonegap，还有更加有名的一站式开发框架Ionicframework  |
| P2P    | 区块链开发、BT客户端 |   webtorrent / ipfs    |
| 编辑器   |   Atom 和 VSCode 都是基于 electron 模块的   |   electron   |
| 物联网与硬件   |    ruff.io和很多硬件都支持node sdk    |  ruff  |

Node.js 应用场景非常丰富，比如 Node.js 可以开发操作系统，但一般我都不讲的，就算说了也没多大意义，难道大家真的会用吗？一般，我习惯将 Node.js 应用场景分为7个部分。

1）初衷，server端，不想成了前端开发的基础设施
2）命令行辅助工具，甚至可以是运维
3）移动端：cordova，pc端：nw.js和electron
4）组件化，构建，代理
5）架构，前后端分离、api proxy
6）性能优化、反爬虫与爬虫
7) 全栈最便捷之路

## Node核心：异步流程控制

Node.js是为异步而生的，它自己把复杂的事儿做了（高并发，低延时），交给用户的只是有点难用的Callback写法。也正是坦诚的将异步回调暴露出来，才有更好的流程控制方面的演进。也正是这些演进，让Node.js从DIRT（数据敏感实时应用）扩展到更多的应用场景，今天的Node.js已经不只是能写后端的JavaScript，已经涵盖了所有涉及到开发的各个方面，而Node全栈更是热门种的热门。

直面问题才能有更好的解决方式，Node.js的异步是整个学习Node.js过程中重中之重。

**1) 异步流程控制学习重点**
**2）Api写法：Error-first Callback 和 EventEmitter**
**3）中流砥柱：Promise**
**4）终极解决方案：Async/Await**

## 异步流程控制学习重点

我整理了一张图，更直观一些。从09年到现在，8年多的时间里，整个Node.js社区做了大量尝试，其中曲折足足够写一本书的了。大家先简单了解一下。

![Node3](https://i5ting.github.io/How-to-learn-node-correctly/media/14913280187332/Screen%20Shot%202017-04-05%20at%2008.43.08.png)

红色代表Promise，是使用最多的，无论async还是generator都可用
蓝色是Generator，过度货
绿色是Async函数，趋势

*结论*：Promise是必须会的，那你为什么不顺势而为呢？

*推荐*：使用Async函数 + Promise组合，如下图所示。

其实，一般使用是不需要掌握上图中的所有技术的。对于初学者来说，先够用，再去深究细节。所以，精简一下，只了解3个就足够足够用了。
![Node4](https://i5ting.github.io/How-to-learn-node-correctly/media/14913280187332/Screen%20Shot%202017-04-05%20at%2008.43.34.png)

结论

1.Node.js SDK里callback写法必须会的。
2.Node.js学习重点: Async函数与Promise
 2.1中流砥柱：Promise
 2.2终极解决方案：Async/Await

## Api写法：Error-first Callback 和 EventEmitter

### Error-first Callback 定义错误优先的回调写法只需要注意2条规则即可：

1.回调函数的第一个参数返回的error对象，如果error发生了，它会作为第一个err参数返回，如果没有，一般做法是返回null。
2.回调函数的第二个参数返回的是任何成功响应的结果数据。如果结果正常，没有error发生，err会被设置为null，并在第二个参数就出返回成功结果数据。

下面让我们看一下调用函数示例，Node.js 文档里最常采用下面这样的回调方式：

```javascript
    function(error,res){
        //process the error and result
    }
```

这里的 callback 指的是带有2个参数的函数："err"和 "res"。语义上讲，非空的“err”相当于程序异常；而空的“err”相当于可以正常返回结果“res”，无任何异常。

### EventEmitter

事件模块是 Node.js 内置的对观察者模式“发布/订阅”（publish/subscribe）的实现，通过EventEmitter属性，提供了一个构造函数。该构造函数的实例具有 on 方法，可以用来监听指定事件，并触发回调函数。任意对象都可以发布指定事件，被 EventEmitter 实例的 on 方法监听到。

在node 6之后，可以直接使用require('events')类

```javascript
var Emitter = require('events');
var uilt = require('uilt');

var myEmitter = function(){
    util.inherits(MyEmitter, EventEmitter)

    const myEmitter = new MyEmitter();

    myEmitter.on('event', (a, b) => {
    console.log(a, b, this);
        // Prints: a b {}
    });

    myEmitter.emit('event', 'a', 'b');
}
```

和jquery、vue里的Event是非常类似的。而且前端自己也有EventEmitter。

### 如何更好的查Node.js文档

API是应用程序接口Application Programming Interface的简称。从Node.js异步原理，我们可以知道，核心在于 Node.js SDK 中API调用，然后交由EventLoop（Libuv）去执行，所以我们一定要熟悉Node.js的API操作。

Node.js的API都是异步的，同步的函数是奢求，要查API文档，在高并发场景下慎用。

笔者推荐使用 <font color = 'blue'>Dash</font> 或 <font color = 'blue'>Zeal</font> 查看离线文档，经常查看离线文档，对Api理解会深入很多，比IDE辅助要好，可以有效避免离开IDE就不会写代码的窘境。

## 中流砥柱：Promise

Node.js 因为采用了错误优先的回调风格写法，导致sdk里导出都是回调函数。如果组合调用的话，就会特别痛苦，经常会出现回调里嵌套回调的问题，大家都非常厌烦这种写法，称之为Callback Hell，即回调地狱。一个经典的例子来自著名的Promise模块q文档里。

```javascript
step1(function (value1) {
    step2(value1, function(value2) {
        step3(value2, function(value3) {
            step4(value3, function(value4) {
                // Do something with value4
            });
        });
    });
});
```

这里只是做4步，嵌套了4层回调，如果更多步骤呢？很多新手浅尝辄止，到这儿就望而却步，粉转黑。这明显不够成熟，最起码你要看看它的应对解决方案吧！

Node.js 约定所有Api都采用错误优先的回调方式，这部分场景都是大家直接调用接口，无太多变化。而Promise是对回调地狱的思考，或者说是改良方案。目前使用非常普遍，可以说是在async函数普及之前唯一一个通用性规范，甚至 Node.js 社区都在考虑 Promise 化，可见其影响之大。

Promise意味着[许愿|承诺]一个还没有完成的操作，但在未来会完成的。与Promise最主要的交互方法是通过将函数传入它的then方法从而获取得Promise最终的值或Promise最终最拒绝（reject）的原因。要点有三个：

递归，每个异步操作返回的都是promise对象
状态机：三种状态转换，只在promise对象内部可以控制，外部不能改变状态
全局异常处理

### 定义

```javascript
var promise = new promise(function(resolve,reject){
    // do a thing, possibly async, then…
    if (/* everything turned out fine */) {
    resolve("Stuff worked!"); //成功返回
  }
  else {
    reject(Error("It broke"));//失败返回
  }
})
```

每个Promise定义都是一样的，在构造函数里传入一个匿名函数，参数是resolve和reject，分别代表成功和失败时候的处理。

### 调用

```javascript
promise.then(function(text){
    console.log(text)// Stuff worked!
    return Promise.reject(new Error('我是故意的'))
}).catch(function(err){
    console.log(err)
})
```

它的主要交互方式是通过then函数，如果Promise成功执行resolve了，那么它就会将resolve的值传给最近的then函数，作为它的then函数的参数。如果出错reject，那就交给catch来捕获异常就好了。

Promise 的最大优势是标准化，各类异步工具库都按照统一规范实现，即使是async函数也可以无缝集成。所以用 Promise 封装 API 通用性强，用起来简单，学习成本低。在async函数普及之前，绝大部分应用都是采用Promise来做异步流程控制的，所以掌握Promise是Node.js学习过程中必须要掌握的重中之重。

**Bluebird**是 Node.js 世界里性能最好的Promise/a+规范的实现模块，Api非常齐全，功能强大，是原生Promise外的不二选择。

>好处如下：
1.避免Node.js内置Promise实现 问题，使用与所有版本兼容
2.避免Node.js 4曾经出现的内存泄露问题
3.内置更多扩展，timeout、 promisifyAll等，对Promise/A+规范提供

推荐学习资料

Node.js最新技术栈之Promise篇 https://cnodejs.org/topic/560dbc826a1ed28204a1e7de
理解 Promise 的工作原理 https://cnodejs.org/topic/569c8226adf526da2aeb23fd
Promise 迷你书 http://liubin.github.io/promises-book/

## 终极解决方案：Async/Await

Async/Await是异步操作的终极解决方案，Koa 2在node 7.6发布之后，立马发布了正式版本，并且推荐使用async函数来编写Koa中间件。

这里给出一段Koa 2应用里的一段代码

```javascript
exports.list = async (ctx, next) => {
  try {
    let students = await Student.getAllAsync();

    await ctx.render('students/index', {
      students : students
    })
  } catch (err) {
    return ctx.api_error(err);
  }
};
```

>它做了3件事儿

1.通过await Student.getAllAsync();来获取所有的students信息。
2.通过await ctx.render渲染页面
3.由于是同步代码，使用try/catch做的异常处理

Async函数要点如下：

Async函数语义上非常好
Async不需要执行器，它本身具备执行能力，不像Generator需要co模块
Async函数的异常处理采用try/catch和Promise的错误处理，非常强大
Await接Promise，Promise自身就足够应对所有流程了，包括async函数没有纯并行处理机制，也可以采用Promise里的all和race来补齐
Await释放Promise的组合能力，外加co和Promise的then，几乎没有不支持的场景

## NodeJs框架分类

|框架名称| 特性 |点评|
| --------   | -----:  | :----:  |
|Express|简单、实用，路由中间件等五脏俱全|最著名的Web框架|
|Derby.js && Meteor|同构|前后端都放到一起，模糊了开发便捷，看上去更简单，实际上上对开发来说要求更高|
|Sails、Total|面向其他语言，Ruby、PHP等|借鉴业界优秀实现，也是 Node.js 成熟的一个标志|
|MEAN.js|面向架构|类似于脚手架，又期望同构，结果只是蹭了热点|
|Hapi和Restfy|面向Api && 微服务|移动互联网时代Api的作用被放大，故而独立分类。尤其是对于微服务开发更是利器
|ThinkJS|面向新特性|借鉴ThinkPHP，并慢慢走出自己的一条路，对于Async函数等新特性支持，无出其右，新版v3.0是基于Koa |v2.0的作为内核的|
|Ko|专注于异步流程改进|下一代Web框架|
|Egg|基于Koa，在开发上有极大便利|企业级Web开发框架|

`对于框架选型`

业务场景、特点，不必为了什么而什么，避免本末倒置
自身团队能力、喜好，有时候技术选型决定团队氛围的，需要平衡激进与稳定
出现问题的时候，有人能够做到源码级定制。Node.js 已经有8年历史，但模块完善程度良莠不齐，如果不慎踩到一个坑里，需要团队在无外力的情况能够搞定，否则会影响进度

`Web编程核心`

异步流程控制（前面讲过了）
基本框架 Koa或Express，新手推荐Express，毕竟资料多，上手更容易。如果有一定经验，推荐Koa，其实这些都是为了了解Web编程原理，尤其是中间件机制理解。
数据库 mongodb或mysql都行，mongoose和Sequelize、bookshelf，TypeOrm等都非常不错。对于事务，不是Node.js的锅，是你选的数据库的问题。另外一些偏门，想node连sqlserver等估计还不成熟，我是不会这样用的。
模板引擎， ejs，jade，nunjucks。理解原理最好。尤其是extend，include等高级用法，理解布局，复用的好处。其实前后端思路都是一样的

作者:狼叔
[原文地址](https://i5ting.github.io/How-to-learn-node-correctly/#1)
