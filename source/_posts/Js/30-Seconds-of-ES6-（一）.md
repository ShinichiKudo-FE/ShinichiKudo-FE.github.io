---
title: 30 Seconds of ES6 （一）
date: 2019-01-27 22:26:07
tags: "Es6"
categories: "Js"
---

# 项目背景简介

[30 seconds of code](https://github.com/30-seconds/30-seconds-of-code)  是一个非常优质精选的 JavaScript 项目 ，总结了大量的使用 ES6  语法实现的代码块，项目的设计目标就是更简洁，更高效，更快速的实现基础代码模块，碎片化学习实用干货， 30 秒掌握一个高质量 ES6 代码块 。

## 学习初衷

学习 ES6 基础知识，提升程序算法能力；学习 JavaScript 基础从 API 开始。
每篇精选 5 段优秀代码块，和 5 个以上 API ，为前端大全栈打下坚实根基！

## 学习方法

认真解读英文版 30 seconds of code 的每个 ES6 代码块，并把其中的 API 重点分析。
认识新单词，提升英语水平，目标可以轻松翻阅英文版专业书，全面提升前端技术软实力。

### Adapter 适配器 ary

创建一个接受最多 n 个参数的函数，忽略任何其他参数。使用 `Array.prototype.slice（0，n）` 方法和 spread 扩展运算符 （…） 调用提供的函数 fn （最多 n 个参数）。

```js
const ary = (fn , n) => (...args) => fn(...args.slice(0, n));
const firstTwoMax = ary(Math.max, 2);
console.log([[2, 6, 'a'], [8, 4, 6], [10]].map(x => firstTwoMax(...x)));//[6, 8, 10]
```

### MDN 解析 Array.prototype.slice() 案例
该 slice() 方法返回一个阵列的一部分的一个浅拷贝，到选自新的数组对象 begin 到 end ( end 不包括)。原始数组不会被修改。

```js
var animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];
console.log(animals.slice(2,4));//['camel', 'duck']
console.log(animals);// ["ant", "bison", "camel", "duck", "elephant"]
```

### MDN 解析 Math.max() 案例
该 Math.max() 函数返回零个或多个数字中的最大值。

console.log(Math.max(-10, 20)); //20

### Array   数组   all

如果提供的谓词函数对集合中的所有元素都返回  true ，则返回  true，否则返回 false 。使用   Array.prototype.every（） 测试集合中的所有元素是否基于  fn  返回  true  。省略第二个参数  fn  ，将布尔值用作默认值。

```js
const all = (arr, fn = Boolean) => arr.every(fn);
console.log(all);
console.log(all([4, 2, 3], x => x >1));//true
console.log(all([1, 2, 3]));//true

```

### MDN  解析 Array.prototype.every()  案例

该  every()  方法测试数组中的所有元素是否都通过了由提供的函数实现的测试。对于空数组，任何情况下调用该方法都会返回 true。

```js
console.log([12, 5, 8, 130, 44].every(x => x >= 10));//false
console.log([12, 54, 18, 130, 44].every(x => x >=10));//true
console.log([].every(x => x));//true
```

### Array   数组   allEqual

检查数组中的所有元素是否相等。使用  Array.prototype.every（） 检查数组中的所有元素是否与第一个元素相同。

```js
const allEqual = arr => arr.every( val => val === arr[0]);
console.log(allEqual([1, 2, 3, 4, 5, 6]));//false
console.log(allEqual([1, 1, 1, 1]));//true
```

### Array   数组   any

如果提供的谓词函数对集合中的至少一个元素返回  true  ，则返回  true   ，否则返回  false   。使用 Array.prototype.some（） 测试集合中的任何元素是否基于  fn  返回  true 。省略第二个参数  fn ，将布尔值用作默认值。

```js
const any = (arr, fn = Boolean) => arr.some(fn);+
console.log(any([0, 1, 2, 0], x => x >=2));//true
console.log(any([0, 0, 1, 0]));//true
```

### MDN  解析 Array.prototype.some()  案例
该   some ()   方法测试数组中是否至少有一个元素通过了由提供的函数实现的测试。此方法返回   false   放在空数组上的任何条件。

```js
console.log([2, 5, 8, 1, 4].some(x => x > 10));//false
console.log([12, 5, 8 ,1 , 4].some( x => x >10));//true

```

### Array   数组   arrayToCSV

将二维数组转换为逗号分隔值（  csv  ）字符串。使用 Array.prototype.map () 和  Array.prototype.join（delimiter）  将一维数组组合成字符串。使用   Array.prototype.join（'\n'） （  ，换行符号）两个组合的全行到  CSV  格式的字符串，每排分捡和一个换行符。在第二  omit  参数delimiter ，使用  delimiter（默认）。

```js
const arrayToCSV = (arr, delimiter = ',') => arr.map( v => v.map( x => `"${x}"`).join(delimiter)).join('\n');

console.log(arrayToCSV([['a', 'b'], ['c', 'd']]));
//"a","b" 
//"c","d"

console.log(arrayToCSV([['a', 'b'],['c', 'd']],';'));
//"a","b" 
//"c","d"
```

### MDN  解析 Array.prototype.map()  案例
该   map()   方法创建一个新数组，其结果是在调用数组中的每个元素上调用提供的函数。

```js
var array1 = [1, 4, 9 ,16];
const map1 = array1.map(x => x * 2);
console.log(map1);// [2, 8, 18, 32]
console.log(array1);//[1, 4, 9, 16]
```

### MDN  解析 Array.prototype.join()  案例

该  join()  方法通过连接数组（或类数组对象）中的所有元素（由逗号或指定的分隔符字符串分隔）来创建并返回新字符串。如果数组只有一个项目，那么将返回该项目而不使用分隔符。如果元素是  undefined  或  null ，则将其转换为空字符串。
```js
var elements = ['Fire', 'Wind', 'Rain'];

console.log(console.log(elements.join('-')));//Fire-Wind-Rain
```