---
title: Node学习(四) --处理接收到的POST数据
date: 2019-03-25 19:31:32
tags: "Node"
categories: "Node"
---

POST数据量通常较大，通常不会一次性从客户端发送到服务端，具体每次发送的大小由协议，以及客户端与服务端之间的协商决定。

因此，Nodejs在处理POST数据时，需要通过request对象的data事件，获取每次传输的数据，并在end事件调用时，处理所有获取的数据。

request对象是一个[http.IncomingMessage](https://link.juejin.im/?target=http%3A%2F%2Fnodejs.cn%2Fapi%2Fhttp.html%23http_class_http_incomingmessage) 类，而它实现了[可读流接口](https://link.juejin.im/?target=http%3A%2F%2Fnodejs.cn%2Fapi%2Fstream.html%23stream_class_stream_readable)，因此具有了可读流的data、end等事件。

需要注意的是，data事件中传入的参数是`Buffer`，`Buffer`只是一个二进制的数据，它有可能只是一段字符串数据，也有可能是文件的一部分，所以处理Buffer数据的时候要注意这一点。

```js
const http = require('http')
const querystring = require('querystring')

const server = http.createServer((req, res) => {
  let bufferArray = []  // 用于存储data事件获取的Buffer数据。

  req.on('data', (buffer) => {
    bufferArray.push(buffer)  // 将Buffer数据存储在数组中。
  })

  req.on('end', () => {
    // Buffer 类是一个全局变量，使用时无需 require('buffer').Buffer。
    // Buffer.concat方法用于合并Buffer数组。
    const buffer = Buffer.concat(bufferArray)
    // 已知Buffer数据只是字符串，则可以直接用toString将其转换成字符串。
    const post = querystring.parse(buffer.toString())
    console.log(post)
  })
})

server.listen(8080)
```

## 同时处理GET/POST请求

通常在开发过程中，同一台服务器需要接收多种类型的请求，并区分不同接口，向客户端返回数据。

最常用的方式，**就是对请求的方法、url进行区分判断，获取到每个请求的数据后，统一由一个回调函数进行处理**。

```js
const http = require('http')
const url = require('url')
const querystring = require('querystring')

const server = http.createServer((req, res) => {
  // 定义公共变量，存储请求方法、路径、数据
  const method = req.method
  let path = ''
  let get = {}
  let post = {}

  // 判断请求方法为GET还是POST，区分处理数据
  if (method === 'GET') {
    // 使用url.parse解析get数据
    const { pathname, query } = url.parse(req.url, true)

    path = pathname
    get = query

    complete()
  } else if (method === 'POST') {
    path = req.url
    let arr = []

    req.on('data', (buffer) => {
      // 获取POST请求的Buffer数据
      arr.push(buffer)
    })

    req.on('end', () => {
      // 将Buffer数据合并
      let buffer = Buffer.concat(arr)

      // 处理接收到的POST数据
      post = querystring.parse(buffer.toString())

      complete()
    })
  }

  // 在回调函数中统一处理解析后的数据
  function complete() {
    console.log(method, path, get, post)
  }
})

server.listen(8080)
```

<!-- more -->

## 实现一个带接口请求的简单服务器

虽然当前还未涉及到数据库的知识，但已经可以通过文件读写，实现一个简单的服务器，需求如下：

* 用户通过GET方法请求/reg接口，实现注册流程。
* 用户通过POST方法请求/login接口，实现登录流程。
* 非接口请求则直接返回相应文件。

**实现思路**

* 新建users.json文件，用于存放用户列表数据。
* 新建index.html文件，实现表单及注册、登录的前端请求功能。
* 服务端判断请求路径，决定是前端是通过接口校验用户数据，还是请求HTML文件。
* 若是接口请求，则通过用户列表判断用户状态，实现注册和登录流程。

### 代码及示例

#### html

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
  用户：<input type="text" name="username" id="username"><br/>
  密码：<input type="text" name="password" id="password"><br/>
  <input type="button" value="注册" id="reg">
  <input type="button" value="登录" id="login">
  <script>
    // 注册
    document.querySelector('#reg').addEventListener('click', async function () {
      const response = await fetch(`/reg?username=${document.querySelector('#username').value}&password=${document.querySelector('#password').value}`)
      const result = await response.json()
      console.log(result)
      alert(result.msg)
    })

    // 登录
    document.querySelector('#login').addEventListener('click', async function () {
      const response = await fetch(`/login`, {
        method: 'POST',
        body: JSON.stringify({
          username: document.querySelector('#username').value,
          password: document.querySelector('#password').value
        })
      })
      const result = await response.json()
      console.log(result)
      alert(result.msg)
    })
  </script>
</body>
</html>
```

#### server.js

```js
const http = require('http')
const url = require('url')
const fs = require('fs')
const querystring = require('querystring')

const server = http.createServer((req, res) => {
  // 定义公共变量，存储请求方法、路径、数据
  const method = req.method
  let path = ''
  let get = {}
  let post = {}

  // 判断请求方法为GET还是POST，区分处理数据
  if (method === 'GET') {
    // 使用url.parse解析get数据
    const { pathname, query } = url.parse(req.url, true)

    path = pathname
    get = query

    complete()
  } else if (method === 'POST') {
    path = req.url
    let arr = []

    req.on('data', (buffer) => {
      // 获取POST请求的Buffer数据
      arr.push(buffer)
    })

    req.on('end', () => {
      // 将Buffer数据合并
      let buffer = Buffer.concat(arr)

      // 处理接收到的POST数据
      post = JSON.parse(buffer.toString())

      complete()
    })
  }

  // 在回调函数中统一处理解析后的数据
  function complete() {
    try {
      if (path === '/reg') {
        // 获取get请求数据
        const {
          username,
          password
        } = get

        // 读取user.json文件
        fs.readFile('./users.json', (error, data) => {
          if (error) {
            res.writeHead(404)
          } else {
            // 读取用户数据
            const users = JSON.parse(data.toString())
            const usernameIndex = users.findIndex((item) => {
              return username === item.username
            })

            // 判断用户名是否存在
            if (usernameIndex >= 0) {
              res.write(JSON.stringify({
                error: 1,
                msg: '此用户名已存在'
              }))
              res.end()
            } else {
              // 用户名不存在则在用户列表中增加一个用户
              users.push({
                username,
                password
              })

              // 将新的用户列表保存到user.json文件中
              fs.writeFile('./users.json', JSON.stringify(users), (error) => {
                if (error) {
                  res.writeHead(404)
                } else {
                  res.write(JSON.stringify({
                    error: 0,
                    msg: '注册成功'
                  }))
                }
                res.end()
              })
            }
          }
        })
      } else if (path === '/login') {
        const {
          username,
          password
        } = post

        // 读取users.json
        fs.readFile('./users.json', (error, data) => {
          if (error) {
            res.writeHead(404)
          } else {
            // 获取user列表数据
            const users = JSON.parse(data.toString())
            const usernameIndex = users.findIndex((item) => {
              return username === item.username
            })

            if (usernameIndex >= 0) {
              // 用户名存在，则校验密码是否正确
              if (users[usernameIndex].password === password) {
                res.write(JSON.stringify({
                  error: 0,
                  msg: '登录成功'
                }))
              } else {
                res.write(JSON.stringify({
                  error: 1,
                  msg: '密码错误'
                }))
              }
            } else {
              res.write(JSON.stringify({
                error: 1,
                msg: '该用户不存在'
              }))
            }
          }
          res.end()
        })
      } else {
        // 若不是注册或登录接口，则直接返回相应文件
        fs.readFile(`.${path}`, (error, data) => {
          if (error) {
            res.writeHead(404)
          } else {
            res.write(data)
          }
          res.end()
        })
      }
    } catch (error) {
      console.error(error);
    }
  }
})

server.listen(8080)

```