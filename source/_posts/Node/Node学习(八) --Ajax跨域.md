---
title: Node学习(八)--Ajax跨域
date: 2019-04-20 09:41:27
tags: Node
categories: Node
---

## Ajax跨域问题的产生原因

Ajax请求无法跨域，在前端开发中是个常见问题，它的产生原因是，**在浏览器接收服务端返回数据时，会检查该数据是否和当前发起请求的网址在同一域名下。若不是，则会丢弃该数据，并返回一个跨域错误**。

> 在Ajax请求中，遵循如下流程：
网页提交一个Ajax请求到浏览器，浏览器将请求发至服务器，服务器接收到请求后，返回响应数据给浏览器（服务器通常不会对域名进行区分），浏览器接收到响应数据时，检测返回数据的域名是否和当前页面域名相同，若相同则将数据返回给网页，不同则丢弃数据。

## Ajax跨域的处理方法

既然跨域问题的产生原因在于浏览器的限制，那么网页端在请求时无法主动规避，此时就需要服务端进行处理。

服务端只需要在响应Ajax请求时，在请求头中加入一个`Access-Control-Allow-Origin`属性，并设置为`*`（表示全部域名）或者当前域名就可以让浏览器不再进行限制.

```js
const http = require('http')

const server = http.createServer((req, res) => {
  console.log(req.headers.origin)
  res.setHeader('Access-Control-Allow-Origin', '*')
  res.write(`{"resultCode": "0000", "msg": "success"}`)
  res.end()
})

server.listen(8080)
```

当然在实际项目中，不可以简单地设置`res.setHeader('Access-Control-Allow-Origin', '*')`，而是要通过`req.headers.origin`判断发起请求的域名是否合法，再设置`Access-Control-Allow-Origin`属性，以免出现安全问题。
