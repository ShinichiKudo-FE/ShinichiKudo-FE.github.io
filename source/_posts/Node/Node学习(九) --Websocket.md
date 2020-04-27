---
title: Node学习(九)--Websocket
date: 2019-04-20 09:45:03
tags: Node
categories: Node
---

# WebSocket的优势

1.性能高。

根据测试环境数据的不同，大约会比普通Ajax请求高2-10倍。 HTTP是文本协议，数据量比较大。

而WebSocket是基于二进制的协议，在建立连接时用的虽然是文本数据，但之后传输的都是二进制数据，因此性能比Ajax请求高。

2.双向通信。

如果是普通Ajax请求，需要实时获取数据，只能用计时器定时发送请求，这样会浪费服务器资源和流量。

而通过WebSocket，服务器可以主动向前端发送信息。

3.安全性高

由于WebSocket出现较晚，相比HTTP协议，在安全性上考虑的更加充分。


接下来，尝试用`Socket.io`建立一个基于WebSocket的双向通信。

## Socket.io

`Socket.io`是在使用WebSocket时的一个常用库，它会自动判断在支持WebSocket的浏览器中使用WebSocket，在其他浏览器中，会使用如flash等方式完成通信。

1.操作简单
2.兼容低端浏览器，如IE6
3.自动进行数据解析
4.自动重连 若出现连接断开的情况，WebSocket会进行自动重连。

## 使用Socket.io建立一个WebSocket应用

```js
const http = require('http')
const io = require('socket.io')

// 1. 建立HTTP服务器。
const server = http.createServer((req, res) => {

})

server.listen(8080)

// 2. 建立WebSocket，让socket.io监听HTTP服务器，一旦发现是WebSocket请求，则会自动进行处理。
const ws = io.listen(server)

// 建立连接完成后，触发connection事件。
// 该事件会返回一个socket对象（https://socket.io/docs/server-api/#Socket），可以利用socket对象进行发送、接收数据操作。
ws.on('connection', (socket) => {
  // 根据事件名，向客户端发送数据，数据数量不限。
  socket.emit('msg', '服务端向客户端发送数据第一条', '服务端向客户端发送数据第二条')

  // 根据事件名接收客户端返回的数据
  socket.on('msg', (...msgs) => {
    console.log(msgs)
  })

  // 使用计时器向客户端发送数据
  setInterval(() => {
    socket.emit('timer', new Date().getTime())
  }, 500);
})
```

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
</head>

<body>
  <!-- 引用Socket.io的客户端js文件，由于Socket.io已在服务端监听了HTTP服务器的请求，一旦收到对该文件的请求，则会自动返回该文件，不需要开发人员配置。 -->
  <!-- 该文件在服务端的位置为/node_modules/socket.io/node_modules/socket.io-client/dist/socket.io.js -->
  <script src="http://localhost:8080/socket.io/socket.io.js"></script>
  <script>
    // 与服务器建立WebSocket连接，该连接为ws协议，socket.io不需要担心跨域问题。
    const socket = io.connect('ws://localhost:8080/')

    // 根据事件名，向服务端发送数据，数据数量不限。
    socket.emit('msg', '客户端向服务端发送数据第一条', '客户端向服务端发送数据第二条')

    // 根据事件名接收服务端返回的数据
    socket.on('msg', (...msgs) => {
      console.log(msgs)
    })

    // 接收服务端通过计时器发送来的数据
    socket.on('timer', (time) => {
      console.log(time)
    })
  </script>
</body>

</html>
```
<!-- more -->

用浏览器打开index.html，即可在控制台看到打印的消息。

# 原生实现WebSocket应用

上面使用了Socket.io实现WebSocket，也是开发中常用的方式。

但这样不利于了解其原理，这里使用Nodejs的Net模块和Web端的WebSocket API实现WebSocket服务器。

## 服务端创建一个Net服务器

```js
// 引入net模块
const net = require('net')

// 使用net模块创建服务器，返回的是一个原始的socket对象，与Socket.io的socket对象不同。
const server = net.createServer((socket) => {
  
})

server.listen(8080)
```

## Web端创建一个WebSocket链接

创建一个WebSocket连接，此时控制台的Network模块可以看到一个处于pending状态的HTTP连接。

![socket](https://user-gold-cdn.xitu.io/2019/3/13/16977a451448747d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

这个连接是一个HTTP请求，与普通HTTP请求的请求你头相比，增加了以下内容：

* Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits  // 扩展信息
* Sec-WebSocket-Key: O3PKSb95qaSB7/+XfaTg7Q== // 发送一个Key到服务端，用于校验服务端是否支持WebSocket
* Sec-WebSocket-Version: 13 // WebSocket版本
* Upgrade: websocket  // 告知服务器通信协议将会升级到WebSocket若服务器支持则继续下一步

```js
const ws = new WebSocket('ws://localhost:8080/')
```

## 服务端使用socket.once，触发一次data事件处理HTTP请求头数据

```js
socket.once('data', (buffer) => {
  // 接收到HTTP请求头数据
  const str = buffer.toString()
  console.log(str)
})
```

打印结果如下:

```js
GET / HTTP/1.1
Host: localhost:8080
Connection: Upgrade
Pragma: no-cache
Cache-Control: no-cache
Upgrade: websocket
Origin: file://
Sec-WebSocket-Version: 13
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML,
like Gecko) Chrome/72.0.3626.121 Safari/537.36
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie: _ga=GA1.1.1892261700.1545540050; _gid=GA1.1.774798563.1552221410; io=7X0VY8jhwRTdRHBfAAAB
Sec-WebSocket-Key: JStOineTIKaQskxefzer7Q==
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

将回车符转换为\r\n显示，结果如下：

```js
GET / HTTP/1.1\r\nHost: localhost:8080\r\nConnection: Upgrade\r\nPragma: no-cache\r\nCache-Control: no-cache\r\nUpgrade: websocket\r\nOrigin: file://\r\nSec-WebSocket-Version: 13\r\nUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36\r\nAccept-Encoding: gzip, deflate, br\r\nAccept-Language: zh-CN,zh;q=0.9\r\nCookie: _ga=GA1.1.1892261700.1545540050; _gid=GA1.1.774798563.1552221410; io=7X0VY8jhwRTdRHBfAAAB\r\nSec-WebSocket-Key: dRB1xDJ/vV+IAGnG7TscNQ==\r\nSec-WebSocket-Extensions: permessage-deflate; client_max_window_bits\r\n\r\n
```

## 将请求头字符串转为对象

创建一个parseHeader方法处理请求头。

```js
function parseHeader(str) {
  // 将请求头数据按回车符切割为数组，得到每一行数据
  let arr = str.split('\r\n').filter(item => item)

  // 第一行数据为GET / HTTP/1.1，可以丢弃。
  arr.shift()

  console.log(arr)
  /* 
    处理结果为：

    [ 'Host: localhost:8080',
      'Connection: Upgrade',
      'Pragma: no-cache',
      'Cache-Control: no-cache',
      'Upgrade: websocket',
      'Origin: file://',
      'Sec-WebSocket-Version: 13',
      'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36',
      'Accept-Encoding: gzip, deflate, br',
      'Accept-Language: zh-CN,zh;q=0.9',
      'Cookie: _ga=GA1.1.1892261700.1545540050; _gid=GA1.1.774798563.1552221410; io=7X0VY8jhwRTdRHBfAAAB',
      'Sec-WebSocket-Key: jqxd7P0Xx9TGkdMfogptRw==',
      'Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits' ]
  */

  let headers = {}  // 存储最终处理的数据

  arr.forEach((item) => {
    // 需要用":"将数组切割成key和value
    let [name, value] = item.split(':')

    // 去除无用的空格，将属性名转为小写
    name = name.replace(/^\s|\s+$/g, '').toLowerCase()
    value = value.replace(/^\s|\s+$/g, '')

    // 获取所有的请求头属性
    headers[name] = value
  })

  return headers
}
```

打印结果如下:

```js
{ host: 'localhost',
  connection: 'Upgrade',
  pragma: 'no-cache',
  'cache-control': 'no-cache',
  upgrade: 'websocket',
  origin: 'file',
  'sec-websocket-version': '13',
  'user-agent':
   'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.121 Safari/537.36',
  'accept-encoding': 'gzip, deflate, br',
  'accept-language': 'zh-CN,zh;q=0.9',
  cookie:
   '_ga=GA1.1.1892261700.1545540050; _gid=GA1.1.585339125.1552405260',
  'sec-websocket-key': 'TipyPZNW+KNvV3fePNpriw==',
  'sec-websocket-extensions': 'permessage-deflate; client_max_window_bits' }
```

## 根据请求头参数，判断是否WebSocket请求

根据`headers['upgrade'] !== 'websocket'`，判断该HTTP连接是否可升级为WebSocket，若可以升级，表示为WebSocket请求。
根据`headers['sec-websocket-version'] !== '13'`，判断WebSocket的版本是否为13，以免因为版本不同出现兼容问题。

```js
socket.once('data', (buffer) => {
  // 接收到HTTP请求头数据
  const str = buffer.toString()
  console.log(str)

  // 4. 将请求头数据转为对象
  const headers = parseHeader(str)
  console.log(headers)

  // 5. 判断请求是否为WebSocket连接
  if (headers['upgrade'] !== 'websocket') {
    // 若当前请求不是WebSocket连接，则关闭连接
    console.log('非WebSocket连接')
    socket.end()
  } else if (headers['sec-websocket-version'] !== '13') {
    // 判断WebSocket版本是否为13，防止是其他版本，造成兼容错误
    console.log('WebSocket版本错误')
    socket.end()
  } else {
    // 请求为WebSocket连接时，进一步处理
  }
})

```

## 校验Sec-WebSocket-Key，完成连接

根据协议规定的方式，向前端返回一个请求头，完成建立WebSocket连接的过程。

若客户端校验结果正确，在控制台的Network模块可以看到HTTP请求的状态码变为101 Switching Protocols，同时客户端的ws.onopen事件被触发。

![console](https://user-gold-cdn.xitu.io/2019/3/13/16977a4514369b43?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```js
socket.once('data', (buffer) => {
  // 接收到HTTP请求头数据
  const str = buffer.toString()
  console.log(str)

  // 4. 将请求头数据转为对象
  const headers = parseHeader(str)
  console.log(headers)

  // 5. 判断请求是否为WebSocket连接
  if (headers['upgrade'] !== 'websocket') {
    // 若当前请求不是WebSocket连接，则关闭连接
    console.log('非WebSocket连接')
    socket.end()
  } else if (headers['sec-websocket-version'] !== '13') {
    // 判断WebSocket版本是否为13，防止是其他版本，造成兼容错误
    console.log('WebSocket版本错误')
    socket.end()
  } else {
      // 6. 校验Sec-WebSocket-Key，完成连接
      /* 
        协议中规定的校验用GUID，可参考如下链接：
        https://tools.ietf.org/html/rfc6455#section-5.5.2
        https://stackoverflow.com/questions/13456017/what-does-258eafa5-e914-47da-95ca-c5ab0dc85b11-means-in-websocket-protocol
      */
      const GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
      const key = headers['sec-websocket-key']
      const hash = crypto.createHash('sha1')  // 创建一个签名算法为sha1的哈希对象

      hash.update(`${key}${GUID}`)  // 将key和GUID连接后，更新到hash
      const result = hash.digest('base64') // 生成base64字符串
	  const header = `HTTP/1.1 101 Switching Protocols\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-Websocket-Accept: ${result}\r\n\r\n` // 生成供前端校验用的请求头

      socket.write(header)  // 返回HTTP头，告知客户端校验结果，HTTP状态码101表示切换协议：https://httpstatuses.com/101。
      // 若客户端校验结果正确，在控制台的Network模块可以看到HTTP请求的状态码变为101 Switching Protocols，同时客户端的ws.onopen事件被触发。
      console.log(header)
      
      // 处理聊天数据
    }
  })
```

## 建立连接后，通过data事件接收客户端的数据并处理

连接开始后，可以在控制台的Network模块看到，该连接会一直保留在pending状态，直到连接断开。

![console1](https://user-gold-cdn.xitu.io/2019/3/13/16977a4524cc9dca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

此时可以通过data事件处理客户端的数据，但此时双方通信的数据为二进制，需要按照其格式进行处理后才可以正常使用。

格式如下：

![console2](https://user-gold-cdn.xitu.io/2019/3/13/16977a45146a18d6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

处理收到的数据：

```js
function decodeWsFrame(data) {
  let start = 0;
  let frame = {
    isFinal: (data[start] & 0x80) === 0x80,
    opcode: data[start++] & 0xF,
    masked: (data[start] & 0x80) === 0x80,
    payloadLen: data[start++] & 0x7F,
    maskingKey: '',
    payloadData: null
  };

  if (frame.payloadLen === 126) {
    frame.payloadLen = (data[start++] << 8) + data[start++];
  } else if (frame.payloadLen === 127) {
    frame.payloadLen = 0;
    for (let i = 7; i >= 0; --i) {
      frame.payloadLen += (data[start++] << (i * 8));
    }
  }

  if (frame.payloadLen) {
    if (frame.masked) {
      const maskingKey = [
        data[start++],
        data[start++],
        data[start++],
        data[start++]
      ];

      frame.maskingKey = maskingKey;

      frame.payloadData = data
        .slice(start, start + frame.payloadLen)
        .map((byte, idx) => byte ^ maskingKey[idx % 4]);
    } else {
      frame.payloadData = data.slice(start, start + frame.payloadLen);
    }
  }

  console.dir(frame)
  return frame;
}
```

处理发出的数据：

```js
function encodeWsFrame(data) {
  const isFinal = data.isFinal !== undefined ? data.isFinal : true,
    opcode = data.opcode !== undefined ? data.opcode : 1,
    payloadData = data.payloadData ? Buffer.from(data.payloadData) : null,
    payloadLen = payloadData ? payloadData.length : 0;

  let frame = [];

  if (isFinal) frame.push((1 << 7) + opcode);
  else frame.push(opcode);

  if (payloadLen < 126) {
    frame.push(payloadLen);
  } else if (payloadLen < 65536) {
    frame.push(126, payloadLen >> 8, payloadLen & 0xFF);
  } else {
    frame.push(127);
    for (let i = 7; i >= 0; --i) {
      frame.push((payloadLen & (0xFF << (i * 8))) >> (i * 8));
    }
  }

  frame = payloadData ? Buffer.concat([Buffer.from(frame), payloadData]) : Buffer.from(frame);

  console.dir(decodeWsFrame(frame));
  return frame;
}
```

对聊天数据进行处理：

```js
socket.once('data', (buffer) => {
  // 接收到HTTP请求头数据
  const str = buffer.toString()
  console.log(str)

  // 4. 将请求头数据转为对象
  const headers = parseHeader(str)
  console.log(headers)

  // 5. 判断请求是否为WebSocket连接
  if (headers['upgrade'] !== 'websocket') {
    // 若当前请求不是WebSocket连接，则关闭连接
    console.log('非WebSocket连接')
    socket.end()
  } else if (headers['sec-websocket-version'] !== '13') {
    // 判断WebSocket版本是否为13，防止是其他版本，造成兼容错误
    console.log('WebSocket版本错误')
    socket.end()
  } else {
      // 6. 校验Sec-WebSocket-Key，完成连接
      /* 
        协议中规定的校验用GUID，可参考如下链接：
        https://tools.ietf.org/html/rfc6455#section-5.5.2
        https://stackoverflow.com/questions/13456017/what-does-258eafa5-e914-47da-95ca-c5ab0dc85b11-means-in-websocket-protocol
      */
      const GUID = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11'
      const key = headers['sec-websocket-key']
      const hash = crypto.createHash('sha1')  // 创建一个签名算法为sha1的哈希对象

      hash.update(`${key}${GUID}`)  // 将key和GUID连接后，更新到hash
      const result = hash.digest('base64') // 生成base64字符串
	  const header = `HTTP/1.1 101 Switching Protocols\r\nUpgrade: websocket\r\nConnection: Upgrade\r\nSec-Websocket-Accept: ${result}\r\n\r\n` // 生成供前端校验用的请求头

      socket.write(header)  // 返回HTTP头，告知客户端校验结果，HTTP状态码101表示切换协议：https://httpstatuses.com/101。
      // 若客户端校验结果正确，在控制台的Network模块可以看到HTTP请求的状态码变为101 Switching Protocols，同时客户端的ws.onopen事件被触发。
      console.log(header)
      
      // 7. 建立连接后，通过data事件接收客户端的数据并处理
      socket.on('data', (buffer) => {
        const data = decodeWsFrame(buffer)
        console.log(data)
        console.log(data.payloadData && data.payloadData.toString())

        // opcode为8，表示客户端发起了断开连接
        if (data.opcode === 8) {
          socket.end()  // 与客户端断开连接
        } else {
          // 接收到客户端数据时的处理，此处默认为返回接收到的数据。
          socket.write(encodeWsFrame({ payloadData: `服务端接收到的消息为：${data.payloadData ? data.payloadData.toString() : ''}` }))
        }
      })
    }
  })
```

这样，一个简单的基于WebSocket的聊天应用就创建完成了，在启动服务器后，可以打开index.html看到效果。