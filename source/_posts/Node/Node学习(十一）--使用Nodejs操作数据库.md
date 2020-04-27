---
title: Node学习(十一) --使用Nodejs操作数据库
date: 2019-04-21 21:51:12
tags: "Node"
categories: "Node"
---

## Nodejs操作数据库

Nodejs操作数据库需要用到`mysql`模块，通过`npm i mysql -D`进行安装。

之后可以通过`mysql.createConnection`方法新建一个数据库连接，需要传入的参数有地址、端口、登录名、密码，以及需要连接的数据库。


`mysql.createConnection`会返回一个`connection`对象，可使用`connection.query`方法，传入SQL语句，对数据库进行相应操作。

```js
const mysql = require('mysql')

// 1. 连接服务器
const connection = mysql.createConnection({
  host: 'localhost',  // 地址
  port: 3306,  // 端口，不传则默认3306
  user: 'root',  // 登录名
  password: '',  // 密码
  database: 'test'  // 连接的数据库
})

const username = 'zzz'
const password = '888888'

// 向数据库中插入数据
connection.query(`INSERT INTO user_table (username, password) VALUES ('${username}', '${password}')`, (err, data) => {
  if (err) {
    console.error(err)
  } else {
    console.log(data)
  }
})

// 查询user_table表的数据
db.query(`SELECT * FROM User_table`, (err, data) => {
  if (err) {
    console.error(err)
  } else {
    console.log(data)
  }
})
```

INSERT语句的打印的是插入结果：

```js
OkPacket {
  fieldCount: 0,
  affectedRows: 1,
  insertId: 8,	// 返回了当前插入语句的id，可以用来告知前端插入数据的id
  serverStatus: 2,
  warningCount: 0,
  message: '',
  protocol41: true,
  changedRows: 0 }
```

SELECT语句的打印结果为查询到的所有数据：

```js
[ RowDataPacket { ID: 1, username: 'lee', password: '123456' },
  RowDataPacket { ID: 3, username: 'chen', password: '654321' },
  RowDataPacket { ID: 8, username: 'zzz', password: '888888' } ]

```

但`mysql.createConnection`方法有一个缺陷，就是它一次只能创建一个连接，一旦有多个用户同时请求，就会造成请求排队等待。

因此为了提高性能，可以改用`mysql.createPool`方法，创建一个连接池，它可以创建n个与服务器之间的连接，需要使用时，可以自动从中选取一个空闲的连接使用。

```js
const connection = mysql.createPool({
  connectionLimit: 10,  // 建立的连接数量，默认为10个
  host: 'localhost',  // 地址
  port: 3306,  // 端口，不传则默认3306
  user: 'root',  // 登录名
  password: '',  // 密码
  database: 'test'  // 连接的数据库
})
```
<!-- more -->

## 与HTTP模块配合实现接口

完成了基本操作后，就可以实现一个完整的注册、登录流程

```js
const http = require('http')
const url = require('url')
const fs = require('fs')
const mysql = require('mysql')

// 1. 连接服务器
const connection = mysql.createPool({
  connectionLimit: 10,  // 建立的连接数量，默认为10个
  host: 'localhost',  // 地址
  port: 3306,  // 端口，不传则默认3306
  user: 'root',  // 登录名
  password: '',  // 密码
  database: 'test'  // 连接的数据库
})

// 2. 与HTTP模块配合使用
const server = http.createServer((req, res) => {
  const {
    pathname,
    query
  } = url.parse(req.url, true)

  if (pathname === '/reg') {
    // 获取get请求数据
    const {
      username,
      password
    } = query

    // 校验参数是否正确
    if (!username || !password) {
      res.write(JSON.stringify({
        error: 1,
        msg: '用户名或密码不可为空'
      }))
      res.end()
    } else if (username.length > 32) {
      res.write(JSON.stringify({
        error: 1,
        msg: '用户名的长度不可超过32位'
      }))
      res.end()
    } else if (password.length > 32) {
      res.write(JSON.stringify({
        error: 1,
        msg: '密码的长度不可超过32位'
      }))
    } else {  // 校验通过，开始注册流程
      // 检查用户名是否已存在
      connection.query(`SELECT ID FROM User_table WHERE username='${username}'`, (err, data) => {
        if (err) {
          res.writeHead(500)
          res.end()
        } else {
          if (data.length) {
            res.write(JSON.stringify({
              error: 1,
              msg: '此用户名已被占用'
            }))
            res.end()
          } else {
            // 将用户名和密码插入数据库
            connection.query(`INSERT INTO user_table (username, password) VALUES('${username}', '${password}')`, (err, data) => {
              if (err) {
                res.writeHead(500)
                res.end()
              } else {
                res.write(JSON.stringify({
                  error: 0,
                  msg: '注册成功'
                }))
                res.end()
              }
            })
          }
        }
      })
    }

  } else if (pathname === '/login') {
    let arr = []

    req.on('data', (buffer) => {
      // 获取POST请求的Buffer数据
      arr.push(buffer)
    })

    req.on('end', () => {
      // 将Buffer数据合并
      let buffer = Buffer.concat(arr)

      // 处理接收到的POST数据
      const post = JSON.parse(buffer.toString())

      // 获取post请求数据
      const {
        username,
        password
      } = post

      // 根据用户名查询
      connection.query(`SELECT username, password FROM User_table WHERE username='${username}'`, (err, data) => {
        if (err) {
          console.error(err)
        } else {
          if (!data.length) {
            // 用户不存在
            res.write(JSON.stringify({
              error: 1,
              msg: '用户名或密码错误'
            }))
            res.end()
          } else if (data[0].password !== password) {
            // 密码不正确
            res.write(JSON.stringify({
              error: 1,
              msg: '用户名或密码错误'
            }))
            res.end()
          } else {
            res.write(JSON.stringify({
              error: 0,
              msg: '登录成功'
            }))
            res.end()
          }
        }
      })
    })
  } else {
    // 若请求不为接口，则默认为请求文件
    fs.readFile(`.${pathname}`, (err, buffer) => {
      if (err) {
        res.writeHead(404)
        res.write(buffer)
      } else {
        res.write(buffer)
      }
      res.end()
    })
  }
})

server.listen(8080)

```
