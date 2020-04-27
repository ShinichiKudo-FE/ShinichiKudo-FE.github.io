---
title: Node学习(三) --处理接收到的GET数据
date: 2019-03-19 19:56:34
tags: "Node"
categories: "Node"
---

# 常用请求方式

GET和POST是最常用的HTTP请求方法，除此之外还有DELETE、HEAD、OPTIONS、PUT、TRACE等，但都很少用到。

| GET        | POST   |
| --------   | -----:  |
| 主要用于获取数据     | 主要用于发送数据 | 
| 数据放在HTTP请求Header中，通过URL进行传输，容量<=32k       |   数据放在HTTP请求body中,容量大，通常上限2G   | 

## 处理Get数据

我们可以使用Nodejs自带的url和querystring模块处理接收到的GET数据。

首先新建一个带form表单的HTML文件，讲输入的数据提交到服务器地址：

```html
<form action="http://localhost:8080/login" method="get">
  用户：<input type="text" name="username"><br/>
  密码：<input type="text" name="password"><br/>
  <input type="submit" value="提交">
</form>
```

服务端在接收到请求数据时，可以有3种方式处理数据：

1. 将请求数据中的req.url进行字符串切割，再用querystring模块获取数据。

```js
const [ pathname, queryStr ] = req.url.split('?')
const query = querystring.parse(queryStr)
console.log(pathname, query)

```

2. 用URL构造函数实例化一个url对象，从中获取到pathname和search值，再用querystring模块解析search数据

```js
const url = new URL(`http://localhost:8080${req.url}`)
const { pathname, search } = url
const query = querystring.parse(search.substring(1, url.search.length))
console.log(pathname, query)
```

3. 使用url模块的parse方法，直接解析出数据。

```js
// parse方法第二个参数若传true，则会直接将解析出的query值转为对象形式，否则它只是字符串形式
const { pathname, query } = url.parse(req.url, true)
console.log(pathname, query)

```