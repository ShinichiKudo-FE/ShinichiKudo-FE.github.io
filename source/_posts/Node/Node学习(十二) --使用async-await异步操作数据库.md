---
title: Node学习(十二) --使用async/await异步操作数据库
date: 2019-04-29 15:08:07
tags: "Node"
categories: "Node"
---

上一篇使用Node.js操作数据库，虽然能实现功能，但是异步操作需要不断写回调函数，代码严重冗余，而且阅读困难。

可以使用[co-mysql](https://github.com/haoxins/co-mysql)，将`query`方法该写为返回一个`Promise`，就可以使用`async/await`进行异步处理。

我们可以参考一下它的源码，看看它是如何实现将回调函数转换为

```js
module.exports = wrapper;

var slice = [].slice;

// 该模块导出为一个包装函数，运行它的返回值是一个带有query方法的对象。
function wrapper(client) {
  // client即为mysql.createPool或mysql.createConnection方法创建的连接对象。
  var query = client.query;

  var o = {};

  o.query = function () {
    // 提取调用query方法时，传入的所有参数，将其存入一个数组。
    var args = slice.call(arguments);

    // 创建一个Promise对象，它将作为o.query的返回值，让其支持async/await方式调用。
    var p = new Promise(function (resolve, reject) {
      // 为args增加一个回调函数，它将作为client.query方法的回调函数运行，它接收到结果时，将进行判断运行reject还是resolve
      args.push(function (err, rows) {
        if (err) {
          reject(err);
        } else {
          resolve(rows);
        }
      });

      // 用apply方法，将o.query接收到的参数都传给client.query，它可以等同于
      /* 
        client.query(args[0], function(err, rows) {
          if (err) {
            reject(err);
          } else {
            resolve(rows);
          }
        })
      */
      query.apply(client, args);
    });

    return p;
  };

  return o;
}
```

于是上一篇的代码可以进行如下优化：
<!-- more -->

```js
const http = require('http')
const url = require('url')
const fs = require('fs')
const mysql = require('mysql')
const coMysql = require('co-mysql')

// 1. 连接服务器
const pool = mysql.createPool({
  connectionLimit: 10,  // 建立的连接数量，默认为10个
  host: 'localhost',  // 地址
  port: 3306,  // 端口，不传则默认3306
  user: 'root',  // 登录名
  password: '',  // 密码
  database: 'test'  // 连接的数据库
})

const connection = coMysql(pool)

/* const username = 'lily'
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
connection.query(`SELECT * FROM User_table`, (err, data) => {
  if (err) {
    console.error(err)
  } else {
    console.log(data)
  }
}) */
// more
// 2. 与HTTP模块配合使用
const server = http.createServer(async (req, res) => {
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
      res.end()
    } else {  // 校验通过，开始注册流程
      try {
        // 检查用户名是否已存在
        const data = await connection.query(`SELECT ID FROM User_table WHERE username='${username}'`)

        if (data.length) {
          res.write(JSON.stringify({
            error: 1,
            msg: '此用户名已被占用'
          }))
        } else {
          // 将用户名和密码插入数据库
          await connection.query(`INSERT INTO userddd_table (username, password) VALUES('${username}', '${password}')`)

          res.write(JSON.stringify({
            error: 0,
            msg: '注册成功'
          }))
        }
      } catch (error) {
        console.error(error)
        res.writeHead(500)
      }
      res.end()
    }
  } else if (pathname === '/login') {
    let arr = []

    req.on('data', (buffer) => {
      // 获取POST请求的Buffer数据
      arr.push(buffer)
    })

    req.on('end', async () => {
      // 将Buffer数据合并
      let buffer = Buffer.concat(arr)

      // 处理接收到的POST数据
      const post = JSON.parse(buffer.toString())

      // 获取post请求数据
      const {
        username,
        password
      } = post

      try {
        // 根据用户名查询
        const data = await connection.query(`SELECT username, password FROM User_table WHERE username='${username}'`)

        if (!data.length) {
          // 用户不存在
          res.write(JSON.stringify({
            error: 1,
            msg: '用户名或密码错误'
          }))
        } else if (data[0].password !== password) {
          // 密码不正确
          res.write(JSON.stringify({
            error: 1,
            msg: '用户名或密码错误'
          }))
        } else {
          res.write(JSON.stringify({
            error: 0,
            msg: '登录成功'
          }))
        }
      } catch (error) {
        console.error(error)
        res.writeHead(500)
      }
      res.end()
    })
  } else {
    // 若请求不为接口，则默认为请求文件
    if (pathname !== '/favicon.ico') {
      fs.readFile(`.${pathname}`, (err, buffer) => {
        if (err) {
          res.writeHead(404)
        } else {
          res.write(buffer)
        }
        res.end()
      })
    } else {
      res.writeHead(404)
      res.end()
    }
  }
})

server.listen(8080)
```