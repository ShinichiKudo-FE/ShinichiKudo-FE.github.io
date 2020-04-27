---
title: 'Node学习(十四) --Node项目-商品列表'
date: 2019-05-25 17:23:20
tags: "Node"
categories: "Node"
---

# Node.js项目介绍

利用学到的知识，实现一个简单但实用的小项目如下：

这是一个商品列表，具有展示商品信息，添加商品，删除商品的功能。

## 项目的文件夹结构

├── package.json
├── server.js # 服务器代码
├── config # 项目配置文件夹
│     ├── config.dev.js # 开发环境配置
│     ├── config.prod.js # 生产环境配置
│     ├── index.js # 导出当前所处环境及配置
├── libs # 项目工具文件夹
│     ├── database.js # 连接数据库
│     ├── http.js # 服务器配置
│     ├── router.js # 处理路由
├── router # 项目路由配置文件夹
│     ├── index.js # 连接数据库
│     ├── list.js # 获取商品列表接口配置
│     ├── add.js # 增加商品接口配置
│     ├── del.js # 删除商品接口配置
├── static # 静态资源文件夹
│     ├── index.html # 前端HTML页面
│     ├── js # 前端JavaScript文件夹
│     ├── css # 前端CSS文件夹
│     ├── fonts # 前端字体文件夹
│     ├── upload # 前端上传文件夹


## 判断当前所处环境

通常项目在开发环境和生产环境要采用不同的，服务器、账号、域名、端口等配置，如果用人工进行切换操作麻烦且容易出错，因此通常使用环境变量进行判断。

首先引入process模块`const process=require('process')`，该模块提供了当前Node.js进程的信息。

1.可以通过process.env环境变量获取开发环境和生产环境系统等参数差异，如开发环境运行在Windows系统上，而生产环境运行在Linux系统，那么就可以使用`process.env.OS === 'Windows_NT'`判断当前所处的是否开发环境。

```js
const mode = process.env.OS === 'Windows_NT' ? 'env' : 'prod'
```

2.也可以通过package.json中配置的启动命令判断处于开发还是生产环境，如开发环境命令`npm start --dev`和生产环境命令`npm run build --build`。

```js
const mode = process.argv[2] === '--dev' ? 'env' : 'prod'
```

## 初始化开发和生产环境配置

在/config/index.js中，判断所处的环境，并将相应环境的标识和参数作为模块导出。开发过程中，可以直接引用相应的配置使用。

```js
const process = require('process')

// 可以通过开发环境和生产环境系统等参数差异，判断处于哪个环境。
// const mode = process.env.OS === 'Windows_NT' ? 'env' : 'prod'

// 也可以通过package.json中配置的启动命令判断处于开发还是生产环境。
const mode = process.argv[2] === '--dev' ? 'env' : 'prod'

module.exports = {
  mode, // 当前所处环境
  ...(mode === 'env' ? require('./config.dev') : require('./config.prod'))  // 当前环境的配置
}
```

1.以开发环境为例，需要使用的配置为服务器域名、端口号、账号、密码、数据库名，以及HTTP端口、静态文件绝对路径、上传文件保存绝对路径，如下：

```js
module.exports = {
  // 数据库配置
  DB_HOST: 'localhost',
  DB_PORT: 3306,
  DB_USER: 'root',
  DB_PASS: '',
  DB_NAME: 'test',

  // HTTP端口
  HTTP_PORT: 8080,
  // 静态文件绝对路径
  HTTP_ROOT: path.resolve(__dirname, '../static/'),
  // 上传文件保存绝对路径
  HTTP_UPLOAD: path.resolve(__dirname, '../static/upload')
}
```

## 连接数据库

在lib文件夹下，创建database.js，用于连接数据库。

<!-- more -->

```js
// 引入mysql和co-mysql，用于连接数据库
const mysql = require('mysql')
const coMysql = require('co-mysql')

// 引入数据库配置
const {
  DB_HOST,
  DB_PORT,
  DB_USER,
  DB_PASS,
  DB_NAME
} = require('../config')

// 1. 创建服务器连接池
const pool = mysql.createPool({
  host: DB_HOST,
  port: DB_PORT,
  user: DB_USER,
  password: DB_PASS,
  database: DB_NAME
})

// 2. 使用co-mysql包装连接池，将连接转换为Async/Await异步方式
const connection = coMysql(pool)

// 3. 作为模块导出使用
module.exports = connection
```

创建数据库连接后，可以在server.js中，创建一个数据库连接，并查看item_table表的数据

```js
const connection = require('./lib/database')

;(async () => {
  // 查询item_table表中的数据
  const response = await connection.query(`SELECT * FROM item_table`)
  console.log(response)
})()
```

若正常连接，即可打印数据如下：

```js
[ RowDataPacket { ID: 1, title: '运动服', price: 299, count: 998 } ]ice: 199, count: 999 },
  RowDataPacket { ID: 2, title: '运动裤', price: 299, count: 998 } ]
```

## 创建路由

在之前的例子中，我们总是要通过if else语句来判断请求的接口路径，并进行相应操作。

这样会极大地降低开发效率，也不利于后期代码维护。

因此，通常的开发中，都会使用路由对不同的接口进行操作。

现在我们就来自己实现一个简单的路由：


**1.先创建一个router对象，用于存储路由表。其中有2个属性，分别为get、post，分别用于存储相应get、post请求地址的回调方法**

```js
// 创建路由表
let router = {
  // 存储get请求的路由
  get: {

  },
  // 存储post请求的路由
  post: {

  }
}
```

**2. 创建一个addRouter方法，用于添加路由配置，参数method为请求方法，url为请求地址，callback为处理该请求的回调函数。**

```js
// 添加路由的方法，method为请求方法，url为请求地址，callback为处理该请求的回调函数
function addRouter(method, url, callback) {
  // 为便于处理，将method和url统一转换成小写
  method = method.toLowerCase()
  url = url.toLowerCase()

  // 将处理请求的回调函数，按方法名和地址储存到路由表中
  router[method][url] = callback
}
```

**3. 创建一个findRouter方法，用于查找相应路由的回调函数。参数method为请求方法，url为请求地址，返回处理路由的回调函数。**

```js
// 查找处理请求的回调函数的方法，method为请求方法，url为请求地址，返回处理路由的回调函数
function findRouter(method, url) {
  // 为便于处理，将method和url统一转换成小写
  method = method.toLowerCase()
  url = url.toLowerCase()

  // 找到路由对应的回调函数，不存在则默认返回null
  const callback = router[method][url] || null

  // 将回调函数返回
  return callback
}
```

**4. 将路由配置作为模块导出**

```js
// 将添加路由和查找路由方法导出
module.exports = {
  addRouter,
  findRouter
}
```

## 创建服务器

在实现了路由之后，就可以以此为基础实现服务器了。

实现服务器分为以下几个步骤：

1.引入所需Node.js模块、服务器配置、路由模块
2.封装统一处理请求数据的方法
3.接收到的请求分为POST请求、GET请求，区分并进行处理
4.POST请求分为数据请求、文件上传请求，区分并进行处理
5.GET请求分为数据请求、读取文件请求，区分并进行处理

接下来，按步骤实现每部分代码。

**1. 引入所需Node.js模块、服务器配置、路由模块**

```js
// 引入创建服务器所需的模块
const http = require('http')
const url = require('url')
const querystring = require('querystring')
const zlib = require('zlib')
const fs = require('fs')
const { Form } = require('multiparty')

// 引入服务器配置
const {
  HTTP_PORT,
  HTTP_ROOT,
  HTTP_UPLOAD
} = require('../config')

// 引入路由模块的查找路由方法
const { findRouter } = require('./router')

const server = http.createServer((req, res) => {
  // 服务器代码
})

// 监听配置的端口
server.listen(HTTP_PORT)
// 打印创建服务器成功信息
console.log(`Server started at ${HTTP_PORT}`)
```

**2. 封装统一处理请求数据的方法**

要处理所有请求接口，需要的参数为method（请求方法）、pathname（请求接口路径）、query（query数据）、post（post数据）、files（文件数据）。

首先，根据method（请求方法）、pathname（请求接口路径），获取在路由配置时，已经配置好的相应接口的回调函数。
其次，若回调函数存在，则直接将参数传入回调函数处理。
最后，若回调函数不存在，则默认为请求一个静态文件，即可将文件读取之后发送给前端。

```js
// 引入创建服务器所需的模块
...

// 引入服务器配置
...

// 引入路由模块的查找路由方法
...

const server = http.createServer((req, res) => {
  // 通过路由处理请求数据的公共方法
  async function processData(method, pathname, query, post, files) {
    const callback = findRouter(method, pathname)  // 获取处理请求的回调函数

    // 若回调函数存在，则表示路由有配置相应的数据处理，即该请求不是获取静态文件。
    if (callback) {
      try {
        // 根据路由处理接口数据
        await callback(res, query, post, files)
      } catch (error) {
        // 出现错误的处理
        res.writeHead(500)
        res.write('Internal Server Error')
        res.end()
      }
    } else {
      // 若回调函数不存在，则表示该请求为请求一个静态文件，如html、css、js等
      ...
    }
  }
})

// 监听配置的端口
server.listen(HTTP_PORT)
// 打印创建服务器成功信息
console.log(`Server started at ${HTTP_PORT}`)
```

**3. 接收到的请求分为POST请求、GET请求，区分并进行处理**

根据请求的method，将请求分为POST请求、GET请求。

若为POST请求，则需要进一步判断是普通数据请求，还是文件请求，并分别进行处理。

而GET请求，只需要将数据传入processData方法进行处理，在processData方法中，区分GET请求获取数据，还是获取静态文件。

```js
// 引入创建服务器所需的模块
...

// 引入服务器配置
...

// 引入路由模块的查找路由方法
...

const server = http.createServer((req, res) => {
  // 解析请求数据
  // 获取请求路径及query数据
  const method = req.method
  const {
    pathname,
    query
  } = url.parse(req.url, true)

  // 处理POST请求
  if (method === 'POST') {
    // POST请求分为数据请求、文件上传请求，区分并进行处理
    ...
  } else {  // 处理GET请求
    // 通过路由处理数据，因为此时是GET请求，只有query数据
    processData(method, url, query, {}, {})
  }

  // 通过路由处理请求数据的公共方法
  async function processData(method, pathname, query, post, files) {
    ...
  }
})

// 监听配置的端口
server.listen(HTTP_PORT)
// 打印创建服务器成功信息
console.log(`Server started at ${HTTP_PORT}`)
```

**4. POST请求分为数据请求、文件上传请求，区分并进行处理**

判断请求头的content-type为`application/x-www-form-urlencoded`时，表示该请求只是单纯传输数据，可以直接当做字符串处理。
若请求头的content-type不对，则表示该请求是上传文件，可以用multiparty进行处理。

```js
// 引入创建服务器所需的模块
...

// 引入服务器配置
...

// 引入路由模块的查找路由方法
...

const server = http.createServer((req, res) => {
  // 解析请求数据
  // 获取请求路径及query数据
  const method = req.method
  const {
    pathname,
    query
  } = url.parse(req.url, true)

  // 处理POST请求
  if (method === 'POST') {
    // 根据请求头的content-type属性值，区分是普通POST请求，还是文件请求。
    // content-type为application/x-www-form-urlencoded时，表示是普通POST请求
    // 普通POST请求直接进行处理，文件请求使用multiparty处理
    if (req.headers['content-type'].startsWith('application/x-www-form-urlencoded')) {
      // 普通POST请求
      let arr = []  // 存储Buffer数据

      // 接收数据
      req.on('data', (buffer) => {
        arr.push(buffer)
      })

      // 数据接收完成
      req.on('end', () => {
        const data = Buffer.concat(arr)  // 合并接收到的数据
        const post = querystring.parse(data.toString()) // 将接收到的数据转换为JSON

        // 通过路由处理数据，因为此时是普通POST请求，不存在文件数据
        processData(method, pathname, query, post, {})
      })
    } else {
      // 文件POST请求
      const form = new Form({
        uploadDir: HTTP_UPLOAD  // 指定文件存储目录
      })

      // 处理请求数据
      form.parse(req)

      let post = {} // 存储数据参数
      let files = {}  // 存储文件数据

      // 通过field事件处理普通数据
      form.on('field', (name, value) => {
        post[name] = value
      })

      // 通过file时间处理文件数据
      form.on('file', (name, file) => {
        files[name] = file
      })

      // 处理错误
      form.on('error', (error) => {
        console.error(error)
      })

      // 数据传输完成时，触发close事件
      form.on('close', () => {
        // 通过路由处理数据，因为此时是POST文件请求，query、post、files数据都存在
        processData(method, pathname, query, post, files)
      })
    }
  } else {  // 处理GET请求
    // 通过路由处理数据，因为此时是GET请求，只有query数据
    processData(method, url, query, {}, {})
  }

  // 通过路由处理请求数据的公共方法
  async function processData(method, pathname, query, post, files) {
    ...
  }
})

// 监听配置的端口
server.listen(HTTP_PORT)
// 打印创建服务器成功信息
console.log(`Server started at ${HTTP_PORT}`)
```

**5. GET请求分为数据请求、读取文件请求，区分并进行处理**

GET请求可以直接用`processData`方法统一处理，若路由中未配置处理数据的方法，则表示该请求为获取静态文件，需要进行单独处理，否则只需要调用路由配置的回调函数处理即可。

```js
// 引入创建服务器所需的模块
...

// 引入服务器配置
...

// 引入路由模块的查找路由方法
...

const server = http.createServer((req, res) => {
  // 解析请求数据
  // 获取请求路径及query数据
  const method = req.method
  const {
    pathname,
    query
  } = url.parse(req.url, true)

  // 处理POST请求
  if (method === 'POST') {
    ...
  } else {  // 处理GET请求
    // 通过路由处理数据，因为此时是GET请求，只有query数据
    processData(method, url, query, {}, {})
  }

  // 通过路由处理请求数据的公共方法
  async function processData(method, pathname, query, post, files) {
    const callback = findRouter(method, pathname)  // 获取处理请求的回调函数

    // 若回调函数存在，则表示路由有配置相应的数据处理，即该请求不是获取静态文件。
    if (callback) {
      try {
        // 根据路由处理接口数据
        await callback(res, query, post, files)
      } catch (error) {
        // 出现错误的处理
        res.writeHead(500)
        res.write('Internal Server Error')
        res.end()
      }
    } else {
      // 若回调函数不存在，则表示该请求为请求一个静态文件，如html、css、js等
      const filePath = HTTP_ROOT + pathname

      // 检查文件是否存在
      fs.stat(filePath, (error, stat) => {
        if (error) {
          // 出现错误表示文件不存在
          res.writeHead(404)
          res.write('Not Found')
          res.end()
        } else {
          // 文件存在则进行读取
          // 创建一个可读流。
          const readStream = fs.createReadStream(filePath)

          // 创建一个Gzip对象，用于将文件压缩成
          const gz = zlib.createGzip()

          // 向浏览器发送经过gzip压缩的文件，设置响应头，否则浏览器无法识别，会自动进行下载。
          res.setHeader('content-encoding', 'gzip')

          // 将读取的内容，通过gzip压缩之后，在通过管道推送到res中，由于res继承自Stream流，因此也可以接收管道的推送。
          readStream.pipe(gz).pipe(res)

          readStream.on('error', (error) => {
            console.error(error)
          })
        }
      })
    }
  }
})

// 监听配置的端口
server.listen(HTTP_PORT)
// 打印创建服务器成功信息
console.log(`Server started at ${HTTP_PORT}`)
```

## 测试服务器

在server.js中引入封装的http模块：

```js
const http = require('./lib/http')
```

再使用node server.js启动服务器，就可以在浏览器中访问`http://localhost:8080/index.html`，看到html页面

## 添加各接口路由配置

获取商品列表路由回调函数

查询`item_table`表中的商品数据后，返回给前台，并将回调函数作为模块导出。

```js
const connection = require('../lib/database')

module.exports = async (res, query, post, files) => {
  try {
    // 查询商品列表
    const data = await connection.query(`SELECT * FROM item_table`)

    res.writeJson({
      error: 0, // error为0则表示接口正常
      data  // 查询到的商品列表数据
    })
  } catch (error) {
    console.error(error)
    res.writeJson({
      error: 1, // error为1则表示接口出错
      msg: '数据库出错' // 接口的错误信息
    })
  }
  res.end()
}
```

### 添加商品路由回调函数

应禁止query语句使用如下写法，容易造成注入攻击。
`connection.query(INSERT INTO item_table (title, price, count) VALUES('${title}, ${price} ${count}'))`
此时若用户传入参数如下：

`http://localhost:8080/add?title=a')%3B%20DELETE%20FROM%20item_table%3B%20SELECT%20(1&price=19.8&count=200`

就会让服务器执行一个这样的语句：

```js
INSERT INTO item_table (title, price, count) VALUES('a'); DELETE FROM item_table; SELECT ('1', 19.8, 200)
```

其意义为：

1.插入一个虚假数据
2.删除item_table表中所有数据
3.返回一个虚假数据

这样就会导致item_table表中的所有数据被删除。

为防止注入攻击，可以使用占位符?代替需要插入数据库的参数，第二个数组参数中的3个值会按顺序填充占位符，该方法可以避免大部分注入攻击，如下：

```js
await connection.query(`INSERT INTO item_table (title, price, count) VALUES(?,?,?)`, [title, price, count])
```

```js
const connection = require('../lib/database')

module.exports = async (res, query, post, files) => {
  let {
    title,
    price,
    count
  } = post

  // 判断是否有传参
  if (!title || !price || !count) {
    res.writeJson({
      error: 1,
      msg: '参数不合法'
    })
  } else {
    // 将价格和数量转为数字
    price = Number(price)
    count = Number(count)

    // 判断价格和数量是否非数字
    if (isNaN(price) || isNaN(count)) {
      res.writeJson({
        error: 1,
        msg: '参数不合法'
      })
    } else {
      try {
        // 使用占位符?代替需要插入数据库的参数，第二个数组参数中的3个值会按顺序填充占位符，该方法可以避免大部分注入攻击。
        await connection.query(`INSERT INTO item_table (title, price, count) VALUES(?,?,?)`, [title, price, count])

        res.writeJson({
          error: 0,
          msg: '添加商品成功'
        })
      } catch (error) {
        console.error(error)
        res.writeJson({
          error: 1,
          msg: '数据库内部错误'
        })
      }
    }
  }
  res.end()
}
```

### 删除商品路由回调函数

```js
const connection = require('../lib/database')

module.exports = async (res, query, post, files) => {
  const ID = query.id

  if (!ID) {
    res.writeJson({
      error: 1,
      msg: '参数不合法'
    })
  } else {
    await connection.query(`DELETE FROM item_table WHERE ID=${ID}`)

    res.writeJson({
      error: 0,
      msg: '删除成功'
    })
  }
  res.end()
}
```

### 添加各接口路由配置

在`/router/index.js`中，引用各个接口的配置，并用addRouter方法添加到路由表中，即可在接收到请求时，查找路由并进行处理。

```js
const {
  addRouter
} = require('../lib/router')

// 添加获取商品列表接口
addRouter('get', '/list', require('./list'))

// 添加商品接口
addRouter('post', '/add', require('./add'))

// 删除商品接口
addRouter('get', '/del', require('./del'))
```

### 在主模块中引用路由

在/server.js中，引用router模块，就可以完成整个服务端的配置。

```js
const connection = require('./lib/database')
const http = require('./lib/http')
const router = require('./router')
```

### 完成前端功能

在/static/index.html中，使用jquery为前端页面实现如下功能：

1.显示商品列表
2.添加随机名称、价格、库存的商品
3.删除对应商品

```js
// 查询商品列表的方法
function getList() {
  $.ajax({
    url: '/list',
    dataType: 'json'
  }).then(res => {
    let html = ``

    res.data.forEach((item, index) => {
      html += (
        `
            <tr>
              <td>${item.title}</td>
              <td>￥${item.price}</td>
              <td>${item.count}</td>
              <td>
                <a data-id="${item.ID}" href="#" class="glyphicon glyphicon-trash">删除</a>
              </td>
            </tr>
          `
      )
    })

    $('tbody').html(html)
  });
} getList()

// 点击添加按钮，随机添加一个商品
$('#addBtn').on('click', function () {
  $.ajax({
    url: '/add',
    method: 'post',
    data: {
      title: `test${Math.floor(Math.random() * 100)}`,
      price: Math.floor(Math.random() * 100),
      count: Math.floor(Math.random() * 100)
    }
  })
    .then((response) => {
      getList()
    })
})

// 点击删除按钮
$('tbody').on('click', 'a', function () {
  $.ajax({
    url: '/del',
    data: {
      id: $(this).attr('data-id')
    }
  })
    .then((response) => {
      getList()
    })
})
```