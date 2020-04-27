---
title: Node学习(五) --Node.js的模块化
date: 2019-03-26 18:34:27
tags: "Node"
categories: "Node"
---

# Nodejs定义模块

Nodejs的模块化由于出现的较早，因此它遵循的是`CommonJS`规范，而非ES6的模块化。

在Nodejs的模块化中，最常用到的有`module对象、exports对象、require方法`。

其中module和exports用于输出模块，require用于引用模块。

## 一个简单的模块例子

先新建一个module1.js文件，代码如下：

```js
module.exports.a = 1
module.exports.b = 2

let c = 3
```


在require.js中，引入模块并打印：

```js
const module1 = require('./module1')

console.log(module1)

```

可以看到打印结果：{ a: 1, b: 2 }。

这段代码的含义如下：

 1.module1.js对外输出了module.exports，module.exports为一个对象，它含有a和b属性。
 2.module1.js中虽然定义了变量c，但它只在module1.js这个模块中存在，从外部无法访问。
 3.在require.js中引用module1.js，必须使用相对路径或绝对路径。
 4.若引用时不带路径，而是直接使用模块名称，则会默认引用项目目录下的node_modules文件夹下的模块，如：


```js
const module2 = require('module2')

console.log(module2)  // { a: 1, b: 2 }
```

若此时项目目录下的node_modules文件夹下存在module2.js文件，则会引用该文件。

若不存在，则会查找系统的node_modules文件夹下，即全局安装的模块，是否存在module2。

若还不存在该模块，则会报错。

通过require导入的模块，可以被任意命名，因此写成const a = require('module2')也是可以的。

## module.exports

上面这个例子中的模块导出，还可以省略module，直接写成`exports.a = 1; exports.b = 1`; ...。

但直接使用exports导出，也仅支持这种写法，若写成：

```js
exports = {
  a: 1,
  b: 2
}
```
<!-- more -->
在引用模块时，只能接收到{}，也就是说exports只支持`exports.a = 1`;这样的语法。

如果要将整个模块直接定义为一个对象、函数、变量、类，则需要使用`module.exports = 123`。

```js
module.exports = {
  a: 1,
  b: 2
}

module.exports = 123

module.exports = {
  a: 1,
  b: 2,
  c: 3
}

module.exports = function () {
  console.log('test')
}

module.exports = class {
  constructor(name) {
    this.name = name
  }

  show() {
    console.log(`Show ${this.name}`)
  }
}
```

module.exports可以让模块被赋值成任意类型，但需要注意的是此时module.exports类似于一个模块内的全局变量。

对它的重复赋值，只有最后的值有效，之前的值会直接被覆盖。

在这个例子中，module3.js模块最终导出为一个类。

因此，通常推荐使用module.exports，可以避免出错。