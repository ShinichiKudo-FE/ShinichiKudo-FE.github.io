---
title: Node学习(十三) --Stream流
date: 2019-05-06 15:50:06
tags: "Node"
categories: "Node"
---

## File System的问题

我们通常会使用File System模块对文件进行读取，如下：

```js
fs.readFile('./test.txt', (error, buffer) => {
  if (error) {
    console.error(error)
  } else {
    // 读取文件成功
    res.write(buffer)
  }
})
```

这样操作简单有效，但这也存在一些问题：

1.占用内存
使用fs读取文件，它是一次性将文件的所有内容读取到内存中，再一次性发送到客户端，因此会占用大量内存。

2.资源使用效率低
从磁盘读取文件期间，磁盘处于忙碌状态，而网络处于空闲状态。
磁盘读取完成后，开始发送文件时，情况正相反，网络处于忙碌状态，此时磁盘却处于空闲状态

## Stream流

相比File System，Stream流读取文件是读一份，发一份，Stream流的写入操作也有同样特点，因此可以解决File System在上面提到的2个问题。

接下来实现一个简单的流，将1.txt文件的内容写入到2.txt中：

```js
const fs = require('fs')

// 创建一个可读流。
const readStream = fs.createReadStream('./1.txt')

// 创建一个可写流。
const writeStream = fs.createWriteStream('./2.txt')

// 将可读流读取的数据，通过管道pipe推送到写入流中，即可将1.txt的内容，写入到2.txt中。
readStream.pipe(writeStream)

// 读取出现错误时会触发error事件。
readStream.on('error', (error) => {
  console.error(error)
})

// 写入完成时，触发finish事件。
writeStream.on('finish', () => {
  console.log('finish')
})
```

## 使用Zlib压缩文件

可以使用Zlib模块，配合Stream流，实现文件压缩功能，如下：

```js
const fs = require('fs')
// 引入zlib模块，用于实现压缩功能
const zlib = require('zlib')

// 创建一个可读流。
const readStream = fs.createReadStream('./google.jpg')

// 创建一个可写流。
const writeStream = fs.createWriteStream('./google.jpg.gz')

// 创建一个Gzip对象，用于将文件压缩成.gz文件
const gzip = zlib.createGzip()

// 将可读流读取的数据，先通过管道pipe推送到gzip中，再推送到写入流中。
// 也就是先将可读流的数据压缩，再推送到可写流中。
readStream.pipe(gzip).pipe(writeStream)

// 读取出现错误时会触发error事件。
readStream.on('error', (error) => {
  console.error(error)
})

// 写入完成时，触发finish事件。
writeStream.on('finish', () => {
  console.log('finish')
})
```
<!-- more -->

## 使用流传输文件到前台

学习了流，我们就可以更加高效地将文件传输到前台：

<!-- more -->
```js
const http = require('http')
const zlib = require('zlib')
const url = require('url')
const fs = require('fs')

const server = http.createServer((req, res) => {
  const {
    pathname
  } = url.parse(req.url, true)

// 创建一个可读流。
  const readStream = fs.createReadStream(`./${pathname}`)

  // 创建一个Gzip对象，用于将文件压缩成.gz文件
  const gzip = zlib.createGzip()

  // 将读取的内容，在通过管道推送到res中，该方法不经过压缩
  readStream.pipe(res)

  // 处理可读流报错，防止请求不存在的文件
  readStream.on('error', (error) => {
    console.error(error);
    res.writeHead(404)
    res.write('Not Found')
    res.end()
  })
})

server.listen(8080)
```

但可以看到，在这个例子里，虽然实现了使用流传输文件，但并没有用到gzip压缩，在传输时还是更多地消耗网络资源，接下来可以引入gzip压缩。

## 文件经过gzip压缩后传输到前台

但此时如果只是简单的用`readStream.pipe(gzip).pipe(res)`传输文件，浏览器在访问时无法直接打开，而是会触发文件下载。
这是因为未设置请求头属性`content-encoding`的值，导致浏览器无法识别用gzip压缩过的文件，这就需要修改请求头`res.setHeader('content-encoding', 'gzip')`，让浏览器可以识别。

这样浏览器就可以正常打开文件了，但若浏览器访问的是不存在的文件，浏览器会报错“无法访问此网站”，这是因为请求头属性`content-encoding`已被设置为gzip，但服务端传给浏览器的是Not Found字符串，浏览器无法识别。

此时可以使用`fs.stat`方法，先检查文件是否存在，若不存在则返回Not Found，若存在则继续传输。

```js
const http = require('http')
const zlib = require('zlib')
const url = require('url')
const fs = require('fs')

const server = http.createServer((req, res) => {
  const {
    pathname
  } = url.parse(req.url, true)

  // 文件的相对路径
  const filepath = `./${pathname}`

  // 检查文件是否存在
  fs.stat(filepath, (error, stat) => {
    if (error) {
      console.error(error);
      res.setHeader('content-encoding', 'identity')
      res.writeHead(404)
      res.write('Not Found')
      res.end()
    } else {
      // 创建一个可读流。
      const readStream = fs.createReadStream(filepath)

      // 创建一个Gzip对象，用于将文件压缩成.gz文件
      const gzip = zlib.createGzip()

      // 向浏览器发送经过gzip压缩的文件，设置响应头，否则浏览器无法识别，会自动进行下载。
      res.setHeader('content-encoding', 'gzip')
      // 将读取的内容，通过gzip压缩之后，在通过管道推送到res中，由于res继承自Stream流，因此也可以接收管道的推送。
      readStream.pipe(gzip).pipe(res)

      // 处理可读流报错，防止文件中途被删除或出错，导致报错。
      readStream.on('error', (error) => {
        console.error(error);
        res.setHeader('content-encoding', 'identity')
        res.writeHead(404)
        res.write('Not Found')
        res.end()
      })
    }
  })
})

server.listen(8080)

```

## 启动器

常用的启动器有forever、pm2等，它们主要用在项目部署阶段

1.使应用不间断运行，如果不使用启动器，命令行窗口一旦关闭，或者出现报错，应用就会停止运行，启动器会帮助应用自动重启。
2.若出现服务器重启，启动器会自动启动应用，不需要手动操作。

常用的启动器有`forever、pm2`等，接下来介绍一下forever的使用。

### forever

forever文档可参考：[github.com/foreverjs/f…](https://github.com/foreverjs/forever#readme)

> 使用forever启动一个服务：

* 安装forever：`npm install forever -g`
* 在命令行运行`forever start server.js`，替代`node server.js`命令。
* 命令行窗口提示`info: Forever processing file: server.js`，表示启动成功，此时如果将窗口关闭，应用照样可以访问。
* 如果需要关闭服务，可以运行`forever stop server.js`。还有一个命令是`forever stopall`，停止全部在运行的任务，但使用要慎重。


forever启动时，还可以添加一些配置，例如`forever start xxx.js -l c:/xxx.log -e c:/xxx_err.log -a，forever start xxx.js`表示启动xxx.js。

-l c:/xxx.log表示将log信息输出到c:/xxx.log文件。
-e c:/xxx_err.log表示将错误信息输出到c:/xxx_err.log文件。
-a 表示新的日志添加到旧日志之后，即保留旧日志。