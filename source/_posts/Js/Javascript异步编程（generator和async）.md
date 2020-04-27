---
title: Javascript异步编程(generator和async)
date: 2018-08-20 10:43:27
tags: "Js"
categories: "Js"
---
# 前言

从Promise开始，JavaScript就在引入新功能，来帮助更简单的方法来处理异步编程，帮助我们远离回调地狱。
Promise是下边要讲的`Generator/yield`与`async/await`的基础，希望你已经提前了解了它。

在大概ES6的时代，推出了`Generator/yield`两个关键字，使用`Generator`可以很方便的帮助我们建立一个处理Promise的解释器。

然后，在ES7左右，我们又得到了`async/await`这样的语法，可以让我们以接近编写同步代码的方式来编写异步代码（无需使用`.then()`或者回调函数）。

两者都能够帮助我们很方便的进行异步编程，但同样，这两者之间也是有不少区别的。

## generator

`Generator`是一个函数，可以在函数内部通过yield返回一个值（此时，**Generator函数的执行会暂定，直到下次触发`.next()`**）
创建一个Generator函数的方法是在function关键字后添加`*`标识。

在调用一个Generator函数后，并不会立即执行其中的代码，函数会返回一个Generator对象，通过调用对象的next函数，可以获得yield/return的返回值。
无论是触发了yield还是return，next()函数总会返回一个带有value和done属性的对象。

value为返回值，done则是一个Boolean对象，用来标识Generator是否还能继续提供返回值。

PS: **generator()函数的执行是惰性的，yield后的代码只在触发next()后执行**

```js
function * oddGenerator () {
  yield 1
  yield 3

  return 5
}

let iterator = oddGenerator()

let first = iterator.next()  // { value: 1, done: false }
let second = iterator.next() // { value: 3, done: false }
let third = iterator.next()  // { value: 5, done: true  }
```

<!-- more -->

### next()的参数传递

我们可以在调用next()的时候传递一个参数，可以在上次yield前接收到这个参数：

```js
function * outputGenerator () {
  let ret1 = yield 1
  console.log(`got ret1: ${ret1}`)
  let ret2 = yield 2
  console.log(`got ret2: ${ret2}`)
}

let iterator = outputGenerator()

iterator.next(1)
iterator.next(2) // got ret1: 2
iterator.next(3) // got ret2: 3
```

第一眼看上去可能会有些诡异，为什么第一条log是在第二次调用next时才进行输出的

这就又要说到上边的Generator的实现了，上边说到了，yield与return都是用来返回值的语法。
函数在执行时遇到这两个关键字后就会暂停执行，等待下次激活。

然后`let ret1 = yield 1`，这是一个赋值表达式，也就是说会先执行=右边的部分，在=右边执行的过程中遇到了yield关键字，函数也就在此处暂停了，在下次触发next()时才被激活，此时，我们继续进行上次未完成的赋值语句`let ret1 = XXX`，并在再次遇到yield时暂停。

* 这也就解释了为什么第二次调用next()的参数会被第一次yield赋值的变量接收到

### 用作迭代器使用

因为Generator对象是一个迭代器，所以我们可以直接用于`for...of`循环：

> 但是要注意的是，用作迭代器中的使用，则只会作用于yield,return的返回值不计入迭代

```js
function * oddGenerator () {
  yield 1
  yield 3
  yield 5

  return 'won\'t be iterate'
}

for (let value of oddGenerator()) {
  console.log(value)
}
// > 1
// > 3
// > 5
```

### Generator函数内部的Generator

除了yield语法以外，其实还有一个`yield*`语法，可以粗略的理解为是 **Generator函数版的[...]**
用来展开Generator迭代器的。

```js
function * gen1 () {
  yield 1
  yield* gen2()
  yield 5
}

function * gen2 () {
  yield 2
  yield 3
  yield 4
  return 'won\'t be iterate'
}

for (let value of gen1()) {
  console.log(value)
}
// > 1
// > 2
// > 3
// > 4
// > 5
```

### 模拟实现Promise执行器

然后我们结合着Promise，来实现一个简易的执行器。

最受欢迎的类似的库是： [co](https://www.npmjs.com/package/co)

```js
function run (gen) {
  gen = gen()
  return next(gen.next())

  function next ({done, value}) {
    return new Promise(resolve => {
     if (done) { // finish
       resolve(value)
     } else { // not yet
       value.then(data => {
         next(gen.next(data)).then(resolve)
       })
     }
   })
  }
}

function getRandom () {
  return new Promise(resolve => {
    setTimeout(_ => resolve(Math.random() * 10 | 0), 1000)
  })
}

function * main () {
  let num1 = yield getRandom()
  let num2 = yield getRandom()

  return num1 + num2
}

run(main).then(data => {
  console.log(`got data: ${data}`);
})
```

* 一个简单的解释器的模拟（仅作举例说明）

在例子中，我们约定yield后边的必然是一个Promise函数
我们只看main()函数的代码，使用Generator确实能够让我们让近似同步的方式来编写异步代码
但是，这样写就意味着我们必须有一个外部函数负责帮我们执行main()函数这个Generator，并处理其中生成的Promise，然后在then回调中将结果返回到Generator函数，以便可以执行下边的代码。

## async

我们用作async/awiat重写上面generator的例子：

```js
function getRandom () {
  return new Promise(resolve => {
    setTimeout(_ => resolve(Math.random() * 10 | 0), 1000)
  })
}

async function main () {
  let num1 = await getRandom()
  let num2 = await getRandom()

  return num1 + num2
}
console.log(`got data: ${await main()}`)
```

> 这样看上去，好像我们从Generator/yield换到async/await只需要把*都改为async，yield都改为await就可以了。 所以很多人都直接拿Generator/yield来解释async/await的行为，但这会带来如下几个问题：

* Generator有其他的用途，而不仅仅是用来帮助你处理Promise
* 这样的解释让那些不熟悉这两者的人理解起来更困难（因为你还要去解释那些类似co的库）

async/await是 **处理Promise的一个极其方便的方法，但如果使用不当的话，也会造成一些令人头疼的问题**

### Async函数始终返回一个Promise

一个async函数，无论你return 1或者throw new Error()。
在调用方来讲，接收到的始终是一个Promise对象：

```js
async function throwError () {
  throw new Error()
}
async function returnNumber () {
  return 1
}

console.log(returnNumber() instanceof Promise) // true
console.log(throwError() instanceof Promise)   // true
```

也就是说，无论函数是做什么用的，你都要按照Promise的方式来处理它。

### Await是按照顺序执行的，并不能并行执行

JavaScript是单线程的，这就意味着await一只能一次处理一个，如果你有多个Promise需要处理，则就意味着，你要等到前一个Promise处理完成才能进行下一个的处理，这就意味着，如果我们同时发送大量的请求，这样处理就会非常慢，one by one：

```js
const bannerImages = [];
async function getImageInfo(){
    return bannerImages.map(async banner => await getImageInfo(banner));
}
```

就像这样的四个定时器，我们需要等待4s才能执行完毕：

```js
function delay () {
  return new Promise(resolve => setTimeout(resolve, 1000))
}

let tasks = [1, 2, 3, 4]

async function runner (tasks) {
  for (let task of tasks) {
    await delay()
  }
}

console.time('runner')
await runner(tasks)
console.timeEnd('runner')
```

像这种情况，我们可以通过`Promise.all()`进行如下优化：

```js
function delay () {
  return new Promise(resolve => setTimeout(resolve, 1000))
}

let tasks = [1, 2, 3, 4]

async function runner (tasks) {
  tasks = tasks.map(delay)
  await Promise.all(tasks)
}

console.time('runner')
await runner(tasks)
console.timeEnd('runner')
```

草案中提到过await*，但现在貌似还不是标准，所以还是采用Promise.all包裹一层的方法来实现

我们知道，Promise对象在创建时就会执行函数内部的代码，也就意味着，在我们使用map创建这个数组时，所有的Promise代码都会执行，也就是说，所有的请求都会同时发出去，然后我们通过await Promise.all来监听所有Promise的响应。

## 结论

Generator与async function都是返回一个特定类型的对象：

Generator: 一个类似{ value: XXX, done: true }这样结构的Object
Async: 始终返回一个Promise，使用await或者.then()来获取返回值
Generator是属于生成器，一种特殊的迭代器，用来解决异步回调问题感觉有些不务正业了。。 而async则是为了更简洁的使用Promise而提出的语法，相比Generator + co这种的实现方式，更为专注，生来就是为了处理异步编程。

现在已经是2018年了，async也是用了好久，就让Generator去做他该做的事情吧。。