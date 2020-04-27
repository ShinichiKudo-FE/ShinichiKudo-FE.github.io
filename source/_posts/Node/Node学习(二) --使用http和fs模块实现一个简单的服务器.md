---
title: Node学习(二) --使用http和fs模块实现一个简单的服务器
date: 2019-03-19 19:43:36
tags: "Node"
categories: "Node"
---

# 准备

1.创建一个www目录，存储静态文件1.html、1.jpg。

    * html文件内容如下：

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
  网页内容
  <img src="/1.jpg" alt="">
</body>
</html>
```

2. 预期实现的结果为：
    a. 在浏览器访问`http://localhost:8080/1.html`。
    b. 读取到`www/1.html`，由`HTML`文件发起对`www/1.jpg`的请求。 
    c. 网页中显示`HTML`内容和图片。

4. 使用Nodejs实现服务端代码：


```js
const http = require('http')
const fs = require('fs')

const server = http.createServer((request, response) => {
  console.log(request.url)  // 在request对象中，可以获取请求的URL，通过URL判断请求的资源。
  fs.readFile(`./www${request.url}`, (error, buffer) => { // 根据URL查找读取相应的文件。
    if (error) {  // 若读取错误，则向前端返回404状态码，以及内容Not Found。
      response.writeHead(404)
      response.write('Not Found')
    } else {  // 若读取成功，则向前端返回读取到的文件。
      response.write(buffer)
    }
    response.end()  // 关闭连接。
  })
})

server.listen(8080)
```

## 服务器需要具备的基本功能

1.响应请求
    如上面的例子，可以根据客户端的请求做出回应，如返回静态文件。
2 数据交互
    定义接口，客户端根据接口，与服务端进行数据交互。
    例如在一个购物流程中，客户端向服务端请求商品数据，展现给客户，客户在购买时，客户端将购买的商品信息发送给服务端处理。
    数据库
3.对数据库中存储的数据进行读写操作。
