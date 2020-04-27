---
title: Node学习(一) --NodeJs简介
date: 2019-03-18 18:30:47
tags: "Node"
categories: "Node"
---

## NodeJs简介

* 简单的说Node.js就是运行在服务端的Javascript
* Node.js是一个基于ChromeV8引擎的Javascript的执行环境
* Node.js使用了一个事件驱动，非阻塞式I/O的模型，使其轻量有高效

## NodeJs的应用环境

由于Nodejs目前还不够成熟，因此一般不会用作独立开发，它的主要用途如下

### 中间层

通常在开发应用时，出于安全考虑，后端的主服务器都不会直接暴露给客户端，两端之间通常需要有一个中间层进行通信。

这样做的好处是，如果中间层出现问题，不会影响后端的主服务器。另外，中间层可以做缓存，或者实现一些业务逻辑，起到降低主服务器复杂度，提高性能的作用。

中间层也可以像CDN一样在各处部署，以提高用户的访问效率。

### 小型服务

可以实现一些小型应用，或某个功能模块。

### 工具类

工具类 Nodejs可以用来开发一些实用工具，如Webpack、Gulp等等。

## Nodejs的优势

* Nodejs的语法与前台JavaScript相同，因此便于前端开发入手

* 性能高

* 利于与前端代码结合，例如在做同样一个数据校验时，前后台代码可以共用，不需要单独开发。

## Nodejs的卸载

当需要升级Nodejs时，建议先完全卸载旧版本，特别是全局已下载的依赖，否则有小概率会出现更新版本后，新安装依赖时报错。

> 完整卸载步骤：
通过系统自带卸载工具，卸载Nodejs，之后最好将Nodejs安装目录整个删除。
手动删除安装目录，如C:\Program Files\nodejs目录下的node_modules文件夹。
找到用户目录，如C:\Users\你的用户名，其中如果有node_modules文件夹，则一起删除。

## 启动一个Nodejs服务器

我们可以新建一个server.js文件，在命令行通过node server.js命令，就可以运行一个服务器，在浏览器访问中访问`http://127.0.0.1:3000/`，就可以看到Hello World。

```js
// 引入Nodejs自带的http模块
const http = require('http');
// 引入Nodejs自带的child_process模块
const childProcess = require('child_process');

const hostname = '127.0.0.1'; // 本机地址
const port = 3000; // 端口

// 创建一个服务器
const server = http.createServer((req, res) => {
  res.statusCode = 200; // 设置响应状态码
  res.setHeader('Content-Type', 'text/plain'); // 设置响应头
  res.end('Hello World\n'); // 向前台输出内容
});

// 开启监听
server.listen(port, hostname, () => {
  // 在命令行打印运行结果
  console.log(`Server running at http://${hostname}:${port}/`);
  // 使用默认浏览器打开地址
  childProcess.exec(`start http://${hostname}:${port}/`);
});

```

## response.write

我们使用了http.createServer创建一个服务器，在它的回调函数中，会传入2个参数，分别为request（请求对象）和response（响应对象）。

通常使用`response.write`方法向前端返回数据，该方法可调用多次，返回的数据会被拼接到一起。

需要注意的是，必须调用`response.end`方法结束请求，否则前端会一直处于等待状态，`response.end`方法也可以用来向前端返回数据。

```js
const server = http.createServer((request, response) => {
  response.write('a')
  response.write('b')
  response.write('c')
  response.end('d')
})
```

<!-- mode -->

## File System

File System是Nodejs中用来操作文件的库，可以通过`const fs = require('fs')`引用。

常用的方法有异步文件读取`fs.readFile`、异步文件写入`fs.writeFile`、同步文件读取`fs.readFileSync`、同步文件写入`fs.writeFileSync`。由于同步操作可能会造成阻塞，通常建议使用**异步操作避免**该问题。

### fs.writeFile

fs.writeFile可向文件写入信息，若文件不存在会自动创建。

```js
fs.writeFile('./test.txt', 'test', (error) => {
  if (error) {
    console.log('文件写入失败', error)
  } else {
    console.log('文件写入成功')
  }
})
```

> fs.writeFile的主要参数：

第一个参数为写入的文件路径
第二个参数为写入内容（可为`<string>` | <Buffer> | <TypedArray> | <DataView>）
第三个参数为回调函数，传入数据为error对象，其为null时表示成功。

### fs.readFile

fs.readFile用来读取文件。

```js
fs.readFile('./test.txt', (error, data) => {
  if (error) {
    console.log('文件读取失败', error)
  } else {
    // 此处因确定读取到的数据是字符串，可以直接用toString方法将Buffer转为字符串。
    // 若是需要传输给浏览器可以直接用Buffer，机器之间通信是直接用Buffer数据。
    console.log('文件读取成功', data.toString())
  }
})
```

> fs.readFile主要参数：
第一个参数为读取的文件路径
第二个参数为回调函数。回调函数传入第一个参数为error对象，其为null时表示成功，第二个为数据，可为`<string>` | `<Buffer>`。