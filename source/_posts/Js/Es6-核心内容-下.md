---
title: Es6 核心内容 下
date: 2018-03-30 16:34:55
tags: "ES6"
categories: "Js"
---

## import export
这两个家伙对应的就是es6自己的`module`功能。

我们之前写的Javascript一直都没有模块化的体系，无法将一个庞大的js工程拆分成一个个功能相对独立但相互依赖的小工程，再用一种简单的方法把这些小工程连接在一起。

> 这有可能导致两个问题：
一方面js代码变得很臃肿，难以维护
另一方面我们常常得很注意每个script标签在html中的位置，因为它们通常有依赖关系，顺序错了可能就会出bug

在es6之前为解决上面提到的问题，我们得利用第三方提供的一些方案，主要有两种CommonJS(服务器端)和AMD（浏览器端，如require.js）。

而现在我们有了es6的module功能，它实现非常简单，可以成为服务器和浏览器通用的模块解决方案。

> ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS和AMD模块，都只能在运行时确定这些东西。

### 传统写法
首先我们回顾下require.js的写法。假设我们有两个js文件: index.js和content.js,现在我们想要在index.js中使用content.js返回的结果，我们要怎么做呢？
首先定义：
```javascript
//content.js
define('content.js', function(){
    return 'A cat';
})
```
<!-- more -->
然后require：

```javascript
//index.js
require(['./content.js'], function(animal){
    console.log(animal);   //A cat
})
```

那CommonJS是怎么写的呢？
```javascript
//index.js
var animal = require('./content.js')

//content.js
module.exports = 'A cat'
```

ES6的写法
```javascript
//index.js
import animal from './content'

//content.js
export default 'A cat'
```

## ES6 module的其他高级用法
```javascript
//content.js

export default 'A cat'    
export function say(){
    return 'Hello!'
}    
export const type = 'dog' 
```

上面可以看出，export命令除了输出变量，还可以输出函数，甚至是类（react的模块基本都是输出类）

```javascript
//index.js

import { say, type } from './content'  
let says = say()
console.log(`The ${type} says ${says}`)  //The dog says Hello
```
这里输入的时候要注意：大括号里面的变量名，必须与被导入模块（content.js）对外接口的名称相同。

如果还希望输入content.js中输出的默认`(default)`, 可以写在大括号外面。

```javascript
//index.js

import animal, { say, type } from './content'  
let says = say()
console.log(`The ${type} says ${says} to ${animal}`)  
//The dog says Hello to A cat
```

### 修改变量名
此时我们不喜欢type这个变量名，因为它有可能重名，所以我们需要修改一下它的变量名。在es6中可以用as实现一键换名。
```javascript
//index.js

import animal, { say, type as animalType } from './content'  
let says = say()
console.log(`The ${animalType} says ${says} to ${animal}`)  
//The dog says Hello to A cat
```

### 模块的整体加载
除了指定加载某个输出值，还可以使用整体加载，即用星号`（*）`指定一个对象，所有输出值都加载在这个对象上面。
```javascript
//index.js

import animal, * as content from './content'  
let says = content.say()
console.log(`The ${content.type} says ${says} to ${animal}`)  
//The dog says Hello to A cat
```
通常星号*结合as一起使用比较合适。

## 终极秘籍
考虑下面的场景：上面的`content.js`一共输出了三个变量（default, say, type）,假如我们的实际项目当中只需要用到type这一个变量，其余两个我们暂时不需要。我们可以只输入一个变量：

```javascript
import { type } from './content' 
```
由于其他两个变量没有被使用，我们希望代码打包的时候也忽略它们，抛弃它们，这样在大项目中可以显著减少文件的体积。

[原文地址](https://segmentfault.com/a/1190000004368132)



