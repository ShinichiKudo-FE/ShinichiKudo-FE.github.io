---
title: Node学习(七) --POST文件上传
date: 2019-04-04 10:05:48
tags: "Node"
categories: "Node"
---

## 简单的文件上传例子

处理文件上传数据，也是前后端交互中重要的功能，它的处理方式与数据不同。

接下来，通过一个例子查看服务端接收到的文件上传数据。

首先，在`post_file.html`中，新建一个用与上传文件的表单：

`form`的属性`enctype="multipart/form-data"`代表表单上传的是文件。

`enctype`的默认值为`enctype="application/x-www-form-urlencoded"`表示上传的是数据类型，此时服务端接收到的数据为“username=lee&password=123456&file=upload.txt”。

```js
<form action="http://localhost:8080/upload" method="POST" enctype="multipart/form-data">
  用户：<input type="text" name="username" value="lee"><br/>
  密码：<input type="password" name="password" value="123456"><br/>
  <input type="file" name="file" id=""><br/>
  <input type="submit" value="提交">
</form>
```

其次，在server.js中，查看接收到的表单提交数据：

```js
const http = require('http')

const server = http.createServer((req, res) => {
  let arr = []

  req.on('data', (buffer) => {
    arr.push(buffer)
  })

  req.on('end', () => {
    let buffer = Buffer.concat(arr)

    console.log(buffer.toString())
  })
})

server.listen(8080)
```

最后，在表单中上传/lesson16/upload.txt文件，并查看打印出的结果：

```js
------WebKitFormBoundaryL5AGcit70yhKB92Y
Content-Disposition: form-data; name="username"

lee
------WebKitFormBoundaryL5AGcit70yhKB92Y
Content-Disposition: form-data; name="password"

123456
Content-Disposition: form-data; name="file"; filename="upload.txt"
Content-Type: text/plain

upload
------WebKitFormBoundaryL5AGcit70yhKB92Y--
```

## 文件上传数据分析

通过分析上面这个例子中，服务端接收到的数据，可以得到以下信息：

`.表单上传的数据，被分隔符“------WebKitFormBoundaryL5AGcit70yhKB92Y”隔开，分隔符在每次上传时都不同。分隔符数据可以从`req.headers['content-type']`中获取，如：`const boundary = '--' + req.headers['content-type'].split('; ')[1].split('=')[1]`。

2.前两段数据中，分别可以获取到表单上传的字段名name="username"，以及数据“lee”。

3.第三段数据中，多了一个字段filename="upload.txt"，它表示的是文件的原始名称。以及可以获取到文件类型“Content-Type: text/plain”，表示这是一个文本文件。最后是文件的内容“upload”。

由此可以看出，文件上传数据虽然有些乱，但还是有规律的，那么处理思路就是按照规律，将数据切割之后，取出其中有用的部分。

## 文件上传数据简化

先回顾一下上面的数据，并将回车符标记出来：

```js
------WebKitFormBoundaryL5AGcit70yhKB92Y\r\n
Content-Disposition: form-data; name="username"\r\n
\r\n
lee\r\n
------WebKitFormBoundaryL5AGcit70yhKB92Y\r\n
Content-Disposition: form-data; name="password"\r\n
\r\n
123456\r\n
Content-Disposition: form-data; name="file"; filename="upload.txt"\r\n
Content-Type: text/plain\r\n
\r\n
upload\r\n
------WebKitFormBoundaryL5AGcit70yhKB92Y--
```
<!-- more -->
可以看出，每段数据的结构其实是这样的：

```js
------WebKitFormBoundaryL5AGcit70yhKB92Y\r\nContent-Disposition: form-data; name="username"\r\n\r\nlee\r\
```

将每段上传数据简化如下：

```js
<分隔符>\r\n字段头\r\n\r\n内容\r\n
```

也就是说，整个表单的数据，就是按照这样的数据格式组装而成。

需要注意的是，在表单数据的结尾不再是\r\n，而是“--”。

## 文件上传数据处理步骤

1.用<分隔符>切分数据：

2.删除数组头尾数据：

3.将每一项数据头尾的的\r\n删除：

4.将每一项数据中间的\r\n\r\n删除，得到最终结果：

## Buffer的数据处理

由于文件都是二进制数据，不能直接将其转换为字符串后再进行处理，否则数据会出错，因此要通过Buffer模块进行数据处理操作。

Buffer模块提供了indexOf方法获取Buffer数据中，其参数所在位置的index值。

Buffer模块提供了slice方法，可通过index值切分Buffer数据。

```js
let buffer = Buffer.from('lee\r\nchen\r\ntest')

const index = buffer.indexOf('\r\n')

console.log(index)
console.log(buffer.slice(0, index).toString())

```

可以看到打印结果分别为3和"lee"，也就是说，我们先找到了"\r\n"所在的index为3，之后从Buffer数据的index为0的位置，切割到index为3的位置，得到了正确的结果。

由此，可以封装一个专门用于切割Buffer数据的方法：

```js
module.exports = function bufferSplit(buffer, separator) {
  let result = [];
  let index = 0;

  while ((index = buffer.indexOf(separator)) != -1) {
    result.push(buffer.slice(0, index));
    buffer = buffer.slice(index + separator.length);
  }
  result.push(buffer);

  return result;
}
```

有了bufferSplit方法，就可以正式开始处理数据了。

## 文件上传数据处理

根据上面的思路，就可以实现一个完整的文件上传流程。

```js
const http = require('http')
const fs = require('fs')
const bufferSplit = require('./bufferSplit')

const server = http.createServer((req, res) => {
  const boundary = `--${req.headers['content-type'].split('; ')[1].split('=')[1]}`  // 获取分隔符
  let arr = []

  req.on('data', (buffer) => {
    arr.push(buffer)
  })

  req.on('end', () => {
    const buffer = Buffer.concat(arr)
    console.log(buffer.toString())

    // 1. 用<分隔符>切分数据
    let result = bufferSplit(buffer, boundary)
    console.log(result.map(item => item.toString()))

    // 2. 删除数组头尾数据
    result.pop()
    result.shift()
    console.log(result.map(item => item.toString()))

    // 3. 将每一项数据头尾的的\r\n删除
    result = result.map(item => item.slice(2, item.length - 2))
    console.log(result.map(item => item.toString()))

    // 4. 将每一项数据中间的\r\n\r\n删除，得到最终结果
    result.forEach(item => {
      console.log(bufferSplit(item, '\r\n\r\n').map(item => item.toString()))

      let [info, data] = bufferSplit(item, '\r\n\r\n')  // 数据中含有文件信息，保持为Buffer类型

      info = info.toString()  // info为字段信息，这是字符串类型数据，直接转换成字符串，若为文件信息，则数据中含有一个回车符\r\n，可以据此判断数据为文件还是为普通数据。

      if (info.indexOf('\r\n') >= 0) {  // 若为文件信息，则将Buffer转为文件保存
        // 获取字段名
        let infoResult = info.split('\r\n')[0].split('; ')
        let name = infoResult[1].split('=')[1]
        name = name.substring(1, name.length - 1)

        // 获取文件名
        let filename = infoResult[2].split('=')[1]
        filename = filename.substring(1, filename.length - 1)
        console.log(name)
        console.log(filename)

        // 将文件存储到服务器
        fs.writeFile(`./upload/${filename}`, data, err => {
          if (err) {
            console.log(err)
          } else {
            console.log('文件上传成功')
          }
        })
      } else {  // 若为数据，则直接获取字段名称和值
        let name = info.split('; ')[1].split('=')[1]
        name = name.substring(1, name.length - 1)
        const value = data.toString()
        console.log(name, value)
      }
    })
  })
})

server.listen(8080)
```

## multiparty

我们可以尝试使用第三方库来完成POST请求的处理，如[multiparty](https://github.com/pillarjs/multiparty)。

### multiparty demo

通过如下例子，可以测试一下multiparty的功能。

它会在field事件中，将数据信息的字段名和值返回。在file事件中，将文件的字段名和信息返回。

上传成功后，会在指定的文件夹创建一个上传的文件，并会将文件重命名（如：IqUHkFe0u2h2TsiBztjKxoBR.jpg），以防止重名。

若上传出现失败，已保存的文件会自动删除。

close事件表示表单数据全部解析完成，用户可以在其中处理已经接收到的信息。

```js
const http = require('http')
const multiparty = require('multiparty')

const server = http.createServer((req, res) => {
  const form = new multiparty.Form({
    uploadDir: './upload' // 指定文件存储目录
  })

  form.parse(req) // 将请求参数传入，multiparty会进行相应处理

  form.on('field', (name, value) => { // 接收到数据参数时，触发field事件
    console.log(name, value)
  })

  form.on('file', (name, file, ...rest) => { // 接收到文件参数时，触发file事件
    console.log(name, file)
  })

  form.on('close', () => {  // 表单数据解析完成，触发close事件
    console.log('表单数据解析完成')
  })
})

server.listen(8080)
```

