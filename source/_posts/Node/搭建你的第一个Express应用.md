---
title: 搭建你的第一个Express应用
date: 2018-04-28 11:27:53
tags: "Express"
categories: "Node"
---
# Express介绍

* Express 是一个简洁而灵活的 node.js Web应用框架, 提供一系列强大特性帮助你创建各种Web应用
* 丰富的HTTP工具以及来自Connect框架的中间件随取随用，创建强健、友好的API变得快速又简单
* Express 不对 node.js 已有的特性进行二次抽象，我们只是在它之上扩展了Web应用所需的功能

使用 Express 可以快速地搭建一个完整功能的网站，它有一套健壮的特性，可用于开发单页、多页和混合Web应用。

## 开始

在确认已经安装了node之后([下载](https://nodejs.org/en/#download)), 在你的机器上创建一个目录，让我们来开始你的第一个应用程序吧

```js
//新建一个目录文件夹,初始化一个packjson.json文件
    $mkdir express-demo && cd express-demo  && npm init
```

```js
//安装Express
$ npm install express -g
```

现在我们来写真正的代码了！创建一个名为app.js 或者 server.js的文件，叫什么看你个人喜好了。 载入express 然后使用代码 express()创建一个新的应用程序:
<!-- more -->

```js
var express = require('express');
var app = express();
```

在这个应用程序实例里，你可以通过 app.VERB()定义路由，下面的例子是"GET /"返回 "Hello World" 字符串。 req 和 res 对象是和node原生提供给你的一致的，你也可以执行 res.pipe(), req.on('data', callback) 等任何事情在没有Express的情况下可以做的事情。

Express 给这些对象加了一个封装好的方法，比如 res.send(), 它会帮你设置Content-Length:

```js
app.get('./',function(req,res){
    console.log("GET 请求成功")；
    req.send('Hello Express');
})
```

现在我们通过执行 app.listen() 来绑定并监听连接。 它接受的参数和nodenet.Server#listen()的方法一致:

```js
app.listen(3000,function(){
    console.log('Listening on port %d', server.address().port);
})
```

然后在终端执行`node app.js` 打开3000端口的页面就会看到`Hello Express`了

## 使用express-generator 快速生成一个应用程序

### 安装express-generator

```js
$ npm install express-generator -g
```

这个工具提供了一个非常简单的生成一个程序骨架的功能，但是它也有局限，比如它只支持很少的几个模板引擎。 而事实上Express几乎支持所有的为node所建的模板引擎。 使用 --help查看一下帮助:

```js
Usage: express [options]

Options:

  -h, --help          输出帮助信息
  -V, --version       输出版本号
  -e, --ejs           添加 ejs 模板引擎支持 (默认为jade)
  -H, --hogan         添加 hogan.js模板引擎支持
  -c, --css   样式 <引擎> 支持 (less|stylus) (默认为css)
  -f, --force         强制在非空目录执行
```

### 使用

```js
 $ express myapp
```

生成目录的如下:
![express](http://hexobed.oss-cn-beijing.aliyuncs.com/18-4-28/74845117.jpg)

默认为[jade](http://jade-lang.com/)和`css`的结构

**如果你想生成一个支持[ejs](https://ejs.bootcss.com/), [Stylus](http://stylus-lang.com/)的应用程序，只需要简单的执行下面的命令**：

则生成的目录为
![ejs](http://hexobed.oss-cn-beijing.aliyuncs.com/18-4-28/16498387.jpg)

* app.js：启动文件，或者说入口文件
* package.json：存储着工程的信息及模块依赖，当在 dependencies 中添加依赖的模块时，运行npm install，npm 会检查当前目录下的 package.json，并自动安装所有指定的模块
* node_modules：存放 package.json 中安装的模块，当你在 package.json 添加依赖的模块并安装后，存放在这个文件夹下
* public：存放 image、css、js 等文件
* routes：存放路由文件
* views：存放视图文件或者说模版文件
* bin：存放可执行文件

**执行应用**

```js
cd myapp
npm install
DEBUG=myapp:* npm start
```
打开3000端口就会看到如下页面：
![expressapp](http://hexobed.oss-cn-beijing.aliyuncs.com/18-4-28/44690178.jpg)

这些就是一个简单的应用程序创建和运行的所有步骤。 记住Express没有限定任何的目录结构，这只是一个方便你工作的基本结构。 如果你想得到更多怎么组织目录结构选择，可以查看github上的[示例](https://github.com/expressjs/express/tree/master/examples)。

## 推荐

最后给大家推荐一个更详细的例子：[使用express搭建一个简单的博客](http://ourjs.com/detail/56b2a6f088feaf2d031d2468);
