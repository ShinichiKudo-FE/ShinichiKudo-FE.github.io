---
title: Es6你可能不知道的事---进阶篇
date: 2018-04-12 10:04:52
tags: "Js"
categories: "Js"
---

# 正文

## Module

模块化是一个进行很久的话题，发展历程中出现过很多模式，例如AMD,CommonJS等等。

**Module**是ES6的一个新特性，是语言层面对模块化的支持。

> 与之前模块加载机制不同，Module是动态加载，导入的变量的**只读引用**，而不是拷贝。

```javascript
// 1. export default 可以做默认导出

// a.js
export default 5;      // 默认导出

// b.js
import b, {a} from './a.js';    // 默认导入，不需要加花括号

// 2. 动态的加载机制

// a.js
export let a = 10;
export let b = 10;
export function add() {
  a = 15;
  b = 20;
  return a+b;
};

// b.js
import {a, b, add} from './a.js';
a+b;    // 20
add();  // 35
a+b;    // 35

```

## Symbol

symbol是es6的一个新特性，他有如下特点：

1.symbol是“新”的基本数据类型，从es6开始基本数据类型变成了6个，**undefind,null,number,boolean,string,symbol**
2.symbol可以用作object的*key*
3.symbol存在全局作用域，利用**Symbol.for(key)方法**，可以创建(全局作用域无指定键)或获取全局作用域的symbol；利用**symbol.keyFor(sym)**可以获取指定的symbol的值
4.Javascript内部使用了很多内置symbol值，作为特殊的键，来实现一些内部功能；例如：**symbol.iterator**用于标示对象的迭代器

> “新” 仅仅是针对前端开发人员来说的，其实 Symbol 概念本身已经在 JavaScript 语言内部长时间使用了
<!-- more -->
```javascript
// 1. "Symbol(desc)" 方法用于创建一个新的 symbol，参数 "desc" 仅用做 symbol 的描述，并不用于唯一标示 symbol
Symbol('abc') === Symbol('abc'); // false，'abc'仅作为两个 symbol 的描述信息

// 2. "Symbol.for(key)" 方法，参数 "key" 是用于在全局作用域中标示 symbol 的唯一键，同时也作为该 symbol 的描述信息
Symbol.for('abc') === Symbol('abc'); //true 左侧为创建，右侧为获取
Symbol.for('abc') === Symbol('abc'); //false

// 3. symbol 无法被 for...in 遍历到 (不可枚举)，可以利用 Object.getOwnPropertySymbols 获取
const obj = {
    [Symbol('abc')]:'abc',
    'abc':'abc'
};
for(let i in obj){
    console.log(i);//abc
}
Object.getOwnPropertySymbols(obj);//[Symbol(abc)]
```

## Iterator + for ...of

ES6 中除了新特性外，还有一个新的规范，那就是关于迭代的规范，他包括两部分分别是 “可迭代规范（iterable protocol）” 和 “迭代器规范（iterator protocol）”。任何实现了前者的对象，都可以进行 for…of 循环。

**String,Map,Set,Array**等是原生可迭代对象，因为他们都是在原型(prototype)对象中实现了`Symbol.iterator`键对应的方法。

> for...of是对对象迭代器的遍历（键和值都可以遍历），而for...in是对象可枚举值的遍历

下面用代码来解释一下两个规范：
```javascript
//1.迭代器规范
const iter = {
    conter:0,
    next(){ // 迭代器是实现了 "next()" 函数的对象
        if(++this.conter < 10){
            return{ // 返回一个含有两个键值对的对象，Object {done => boolean, value => any}
                done:false,
                value:this.conter
            }
        }else{
            this.conter = 0;
            return { // done = true 时，value非必须
                done : true;
            }
        }
    }
}
// 2. 可迭代规范，实现 "Symbol.iterator => func()" 键值对；而 "func()" 返回一个 迭代器对象
const iterObj = {};
for(let i of iterObj){};
iterObj[Symbol.iterator] = function(){
    return iter;
}
for(let i of iterObj){
    console.log(i);//1,2,3,4,5,6,7,8,9
}
```

## 关于集合
原来我们使用集合，多数情况下会直接用 Object 代替，ES6新增了两个特性，*Map* 和 *Set*，他们是对 JavaScript 关于集合概念的补充。

### Map
为什么要有Map这种数据结构？直接使用Object不好吗？是不是完全可以使用Map取代object用于数据存取？

> Map和Object的区别？
1.Map和Object都可以用作数据存取，Map适用于存取需要**常需要变化(增减键值对)或遍历的数剧集**，而Object适用于**存取静态（例如配置信息）**的数据集
2.Object的key必须是`String`或者`symbol`类型，而Map却无此限制，可以是`任何值`
3.Map可以很方便的取到键对值的数量，而Object则需要用额外途径


### Set
Set作为最简单的集合，有如下特点：

1.Set可以存储任何类型的值，**遍历顺序与插入顺序相同**

2.Set内**无重复值**

```javascript
// 1. Set 的构造函数可以传入一个 “可迭代的对象（例如数组）”，其中包含任意值
const first = new Set(['a', 1, {'b': 1}, null]);    // Set {"a", 1, Object {b: 1}, null}

// 2. Set 无法插入重复的值
first.add(1);    // Set {"a", 1, Object {b: 1}, null}
```

## WeakMap + WeakSet
`WeakMap` 与 `WeakSet` 作为一个比较新颖的概念，其主要特点在于弱引用。

相比于 Map 与 Set 的强引用，弱引用可以令对象在 “适当” 情况下正确被 GC 回收，减少内存资源浪费。

**但由于不是强引用，所以无法进行遍历或取得值数量，只能用于值的存取（WeakMap）或是否存在值得判断（WeakSet）**

> * 在弱引用的情况下，GC 回收时，不会把其视作一个引用；如果没有其他强引用存在，那这个对象将被回收

```javascript
// 1. WeakMap 键必须是对象
const err = new WeakMap([['a',1]]);    // TypeError: Invalid value used as weak map key

// 2. WeakMap/WeakSet 的弱引用
const wm = new WeakMap([[{'a':1},1]]);    // Object {'a': 1} 会正常被 GC 回收

const ws = new WeakSet(); 
ws.add({'a':1});    // Object {'a': 1} 会正常被 GC 回收

const obj = {'b': 1};
ws.add(obj);        // Object {'b': 1} 不会被正常 GC 回收，因为存在一个强引用
obj = undefined;    // Object {'b': 1} 会正常被 GC 回收
```

## 异步编程
在 ES6 之前，JavaScript 的异步编程都跳不出回调函数这个方式。回调函数方式使用非常简单，在简单异步任务调用时候没有任何问题，但如果出现复杂的异步任务场景时，就显得力不从心了，最主要的问题就是多层回调函数的嵌套会导致代码的横向发展，难以维护；ES6 带来了两个新特性来解决异步编程的难题。
```javascript
// 一个简单的多层嵌套回调函数的例子 (Node.js)
const git = require('shell').git;
const commitMsg = '...';

git.add('pattern/for/some/files/*', (err) => {
  if(!err){
    git.commit(commitMsg, (err) => {
      if(!err){
        git.push(pushOption);
      }else{
        console.log(err);
      }
    });
  }else{
    console.log(err);
  }
});
```

### Promise

Promise是ES6的一个新特性，ta他有如下特点：

1.本质还是回调函数
2.区分成功和失败的回调，省去嵌套在内层的判断逻辑
3.可以很轻松的完成回调函数模式到promise模式的转化
4.代码由回调函数嵌套的横向扩展，变为链式调用的纵向扩展，便于编程和维护

promise 虽然优势颇多，但是代码结构仍与同步代码区别较大

```javascript
// 上例用 Promise 实现
// 假定 git.add, git.commit, git.push 均做了 Promise 封装，返回一个 Promise 对象
const git = require('shell').git;
const commitMsg = '...';

git.add('pattern/for/some/files/*')
  .then(() => git.commit(commitMsg))
  .then(git.push)
  .catch((err) => {
    console.log(err);
  });
```

### generator
Generator 作为 ES6 的新特性，是一个语言层面的升级。它有以下几个特点：
1.可以使用yield关键字，终止执行返回（内到外）
2.可以通过next(val)方法调用重新唤醒，继续执行（外回内）
3.运行时（包括挂起态），**共享局部变量**
4.`Generator` 执行会返回一个结果对象，结果对象本身既是迭代器，同时也是可迭代对象（同时满足两个迭代规范），所以 Generator 可以`直接用于 自定义对象迭代器`

由于具备以上特点（第四点除外），Generator 也是 JavaScript 对 协程（coroutine）的实现，协程可以理解为 “可由开发人员控制调度的多线程”

> * 协程按照调度机制来区分，可以分为对称式和非对称式

> * 非对称式：被调用者（协程）挂起时，必须将控制权返还调用者（协程）

> * 对称式：被调用者（协程）挂起时，可将控制权转给 “任意” 其他协程

> * JavaScript 实现的是 非对称式协程（semi-coroutine）；非对称式协程相比于对称式协程，代码逻辑更清晰，易于理解和维护

协程给 JavaScript 提供了一个新的方式去完成异步编程；由于 Generator 的执行会返回一个迭代器，需要手动去遍历，所以如果要达到自动执行的目的，除了本身语法外，还需要实现一个执行器，例如 `TJ 大神的 co 框架`。

```javascript
// 上例用 Generator 实现
// 假定 git.add, git.commit, git.push 均做了 Promise 封装，返回一个 Promise 对象
const co = require('co');
const git = require('shell').git;

co(function* (){
  const commitMsg = '...';      // 共享的局部变量
  
  yield git.add('pattern/for/some/files/*');
  yield git.commit(commitMsg);
  yield git.push();
}).catch((err) => {
  console.log(err);
});
```

Generator 是一个 ES6 最佳的异步编程选择么？显然不是，因为除了基本语法外，我们还要额外去实现执行器来达到执行的目的，但是它整体的代码结构是优于回调函数嵌套和 Promise 模式的。

### Async-Await

这并不是一个 ES6 新特性，而是 ES7 的语法，放在这里是因为它将是 JavaScript 目前支持异步编程最好的方式
```javascript
// 上例用 async-await 实现
// 假定 git.add, git.commit, git.push 均做了 Promise 封装，返回一个 Promise 对象
const git = require('shell').git;
(async function(){
    const commitMsg = ''; //共享局部变量
    try(){
        await git.add('pattern/for/some/files/*');
        await git.commit(commitMsg);
        await git.push();
    }catch(err){
        console.log(err);
    }
})()
```

## 元编程
元编程是指的是开发人员对 “语言本身进行编程”。一般是编程语言暴露了一些 API，供开发人员来操作语言本身的某些特性。ES6 两个新特性 Proxy 和 Reflect 是 JavaScript 关于对象元编程能力的扩展。

### proxy
Proxy 是 ES6 加入的一个新特性，它可以 `“代理” 对象的原生行为`，替换为`执行自定义行为`。

这样的元编程能力使得我们可以更轻松的扩展出一些特殊对象。

> * 任何对象都可以被“代理”
> * 利用 Proxy.revocable(target, handler) 可以创建出一个可逆的 “被代理” 对象

```javascript
// 简单 element 选择控制工具的实现
const cacheElement = function(target, prop) {
  if(target.hasOwnProperty(prop)){
      return target[prop];
    }else{
      return target[prop] = document.getElementById(prop);
    }
}

const elControl = new Proxy(cacheElement, {
  get: (target, prop) => {
    return cacheElement(target, prop);
  },
  set: (target, prop, val) => {
    cacheElement(target, prop).textContent = val;
  },
  apply: (target, thisArg, args) => {
    return Reflect.ownKeys(target);
  }
});

elControl.first;     // div#first
elControl.second;    // div#second
elControl.first = 5;    // div#first => 5
elControl.second = 10;  // div#second => 10
elControl();    // ['first', 'second']
```

### Reflect
ES6 中引入的 `Reflect` 是另一个元编程的特性，它使得我们可以直接操纵对象的原生行为。`Reflect` 可操纵的行为与 `Proxy` 可代理的行为是一一对应的，这使得可以在 `Proxy` 的自定义方法中方便的使用 `Reflect` 调起原生行为
```javascript
// 1. Proxy 的自定义方法中，通过 Reflect 调用原生行为
const customProxy = new Proxy({
  'custom': 1
}, {
  get: (target, prop) => {
    console.log(`get ${prop} !`);
    return Reflect.get(target, undefined, prop);
  }
});

customProxy.custom;  // get custom, 1

// 2. 与 Object 对象上已经开放的操作原生行为方法相比，语法更加清晰易用（例如：Object.hasOwnProperty 与 Reflect.has）
const symb = Symbol('b');
const a = {
  [symb]: 1,
  'b': 2
};

if(Reflect.has(a, symb) && Reflect.has(a, 'b')){  // good
  console.log('good');
}
Reflect.ownKeys(a);  // ["b", Symbol(b)]
```

## 进阶阅读

如果你关注兼容性，推荐看：https://kangax.github.io/compat-table/es6/，这里介绍了从 ES5 到 ES2016+ 的所有特性（包括仍未定稿的特性）及其在各环境的兼容性

如果你关注性能，推荐看：http://kpdecker.github.io/six-speed/，这里通过性能测试，将 ES6 特性的原生实现与 ES5 polyfill 版本进行对比，覆盖了各主流环境；同时也可以侧面对比出各环境在原生实现上的性能优劣

如果你想全面了解特性，推荐看：https://developer.mozilla.org/en-US/docs/Web/JavaScript，覆盖特性的各方面，包括全面的 API（包括不推荐和废弃的）和基础用法

如果你想看特性更多的使用示例和对应的 polyfill 实现，推荐看：http://es6-features.org/#Constants，这里对各个特性都给出了使用丰富的例子和一个 polyfill 实现，简单明了

如果想了解 ECMA Script 最多最全面的细节，英语又比较过硬，推荐在需要时看：http://www.ecma-international.org/publications/files/ECMA-ST/Ecma-262.pdf，（或者直接看最新的：https://tc39.github.io/ecma262/）


[原文地址](http://taobaofed.org/blog/2016/11/03/es6-advanced/)