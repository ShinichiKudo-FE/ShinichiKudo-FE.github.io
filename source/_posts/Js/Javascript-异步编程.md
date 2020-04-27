---
title: Javascript 异步编程
date: 2018-04-29 22:55:37
tags: "Js"
categories: "Js"
---
# 为什么JavaScript的异步编程经常被人提起，我想到以下两点原因。

首先，JavaScript是单线程的，用事件循环的机制来保证系统的正常运行。如果有同步的ajax请求或者很复杂的运算，JavaScript要等这些操作完成，才能响应其他事件，页面会进入假死状态。
然而对于日渐复杂的web应用来说，这个是致命的。这也是为什么Node.js适合高I/O操作的业务，而像PHP，直到现在，I/O操作也没有提供对应的异步版本，对于PHP来说每个请求都在php-fpm的一个新线程里，这个线程阻塞了不影响其他线程，异步也就没有那么迫切。

其次，JavaScript作为动态语言，函数是一等一的公民，可以随意定义函数，将函数作为变量传值传参，极大的方便了JSer灵活的进行异步编程。在C语言里面，也有异步编程，不管是IOCP模型还是epool模型，他们都是创建多个线程来统一的处理回调，在每个线程函数里用一个死循环，在死循环里用一个阻塞的的操作去等待完成的I/O操作。
这样的话，I/O操作函数和回调函数不在同一个线程里，这个给流程控制带来很大的不便。Node.js底层肯定也是用的这个来处理I/O的，但这对于JSer来说是透明的，不用操心。

好像有点跑偏了，我们来看一下JavaScript中有哪些灵活的异步编程吧。

编程，就是由coder定好代码的执行顺序，让计算机去跑。那对于同步操作，这个过程很简单。比如执行我们有A和B操作，A操作完成再执行B，如下

```js
A();
B();
```

这样就能很好的体现流程，但是当A操作是个异步操作的时候，A操作一开始只执行一个发起的操作（发起请求，发起等待），至于什么时候能完成只有A知道。这时候如果还按如上的方式组织代码，就不能保证“A操作完成再执行B”。

当然，我们可以直接把B直接糅合进A。

```js
function A(){B()}
```

这样代码之间耦合度太高，增加代码复用的难度 ，于是有了callback形式。

<!-- more -->

## callback

> *被广泛吐槽的callback，真正该被吐槽的应该是不合格的JSer吧*。

`setTimeout`就是一个典型的异步操作，延迟时间到了时候只有`setTimeout`自己能得到通知。那么我们就可以以函数的形式先告诉`setTimeout`延迟时间到了时候我们要执行什么操作。

```js
setTimeout(() => {

    console.log(1);

    console.log(2);

}, 1000);
```

我们都用setTimeout来表示异步操作。

```js
function A(callback){
    console.log('do a start');

    setTimeout(() => {

        console.log('do a end');

        callback();

    }, 1000);
}
function B() {

    console.log('do b');

}

A(B);
```

这样保证了“A操作完成再执行B”，也解决了耦合度高的问题。

**但是callback在流程复杂的情况下，难以达到理想编码体验；在流程长的情况下会形成回调地狱，流程难以阅读和维护**。

## Promise

> *callback的问题是下一步的操作不是靠返回值，而是要靠callback来决定的。这个是跟同步操作最大的区别，也是导致callback烦恼的根本原因。*

`Promise`让异步操作的状态返回出来，而不是在内部处理，是链式调用的基础，也给之后同步方式的写法奠定基础。

```js
function A(){
    console.log('do a start');
    return new Promise((resolve,reject) => {
        setTimeout(() => {
            console.log('do a end');
            resolve();
        },1000)
    })
}
function B(){
    console.log('do B');
}
A().then(B)
```

配合`defer`，`Promise.all`和`Promise.race`，使用Promise可以写出很清晰的异步流程。从本质上来讲，**Promise连接了异步操作和操作结果，只是将回调延后，再通过链式调用让流程更清晰**。

## generator

> *调用`generator`函数不立即执行，调用一次`next`执行到下一个`yield`，拿到`yield`右边表达式的值，下次调用next传的值赋值给yield左边。*

很尴尬啊，写完上面一段话，自己再回头看，也不知道说的啥。还是来看代码吧。

```js
function* generator(){
    console.log('start');

    const res = yield 1 + 3;

    console.log(res);

    return res;
}
const gen = generator();

const res1 = gen.next();

console.log(res1);

const res2 = gen.next(5);

console.log(res2);
```

可以看到yield的左值不是等于yield右边的结果，而是等于你下一次调用next的时候传入的值。这个特性，使得`generator配合Promise`，能很好的控制异步。如果yield右边是一个Promise，在Promise的状态变成resolved的时候调用下一次next，传入PromiseValue。

```js
function sleep(msTime = 1000){
    return new Promise((resolve,reject) => {
        setTimeout(resolve,msTime);
    });
}
function* generator() {

    const startTime = Date.now();

    yield sleep(1000);

    console.log(Date.now() - startTime);

}

const gen = generator();

gen.next().value.then((val) => {

    gen.next(val);

});
```

这样所有的流程控制的回调代码都脱离了逻辑函数本身，在`generator`里，同步和异步的代码混在一起了，但是整个写法看上去是一个同步的，异常的清晰。而后面的控制代码，可以写成一个通用的控制函数，让`generator`自执行，这个是`co`模块做的。

```js

import co from 'co';

const doSomeThing = co.wrap(function* generator() {

    const startTime = Date.now();

    yield sleep(1000);

    console.log(Date.now() - startTime);

});

doSomeThing();

```

使用co模块yield右值可以是`promise`和`trunk函数`（为什么是这两个？因为这两个都能从回掉里拿到上一个异步操作的状态），不能是基本类型，确实从异步的角度考虑yield右值放基本类型没有啥意义，但是因为这个报错是有点难以接受的。还值得一提的是co做了一些处理，yield右值也可以是`Array`和`Object`，但是里面的值还是必须是promise和trunk。

## async/await

> *async/`await`是es2017出的规范，可以说co和generator的`语法糖`。不一样的是await的右值**兼容了基本类型**，但是不再处理Array和Object。*

```js
async function doSomeThing() {

    const startTime = Date.now();

    await sleep(1000);

    console.log(Date.now() - startTime);

}

doSomeThing();
```

从`async/await`来看JavaScript的异步编程，可以说是很清楚了，同步和异步无缝连接，写起来行云流水，看来清晰明了。想必大家都想用`async/await`来提高工作效率吧，那么使用async/await要**注意**什么呢？

* 1.`async/await`离不开`Promise`，使用`Promise.all`和`Promise.race`等优化流程。await写起来太方便了，当两个操作可以并行的时候还请用Promise.all，连续两个await是串行的操作。

* 2.async函数始终返回的一个Promise，就算简单如下函数。

```js
async function doSomeThing() {

    return 123;

}
```

* 3.await右边异步操作变成`rejected`的时候抛出错误能被`try/catch`捕捉到，请不要吝啬使用`try/catch`。

* 4.还有一点重要的忘记说了，不支持的环境下请转码。

现在的`async/await`比较完美了，纵观这个变化历程，其实也是蛮辛酸的。在Callback的路上坚持走了很久，期间也出了wind.js等库，或者使用trunk函数，或者使用流式编程的rx.js等。可以看到先辈们的努力，正是他们的努力如今我们可以才可以享受如此硕果。

`trunk函数`(阮一峰老师的介绍)是个很有意思的想法，通过把callback形式的函数柯里化，很简单的就跟promise一样做到了延迟了回掉。那我们是不是也可以简单的用trunk函数来实现链式调用，答案是no。

不怕大家笑话，因为Promise的then强调的要返回一个Promise，所以，在写本文之前我还一直以为Promise链式调用是依赖回掉的返回值。回想起来，有点好笑，在下一个then调用之前，callback都还没有执行，怎么可能依赖他的返回值来链式调用呢。

这让我们回到一个编程的最基本，就是定义好一系列的操作和操作之前的关系。简单的来将，我们可以把这一系列的操作看成一个数组或者说链表可能更贴切一点，是一环扣一环的，程序开始执行，数组第一个操作完成，然后执行第二个操作，依次类推，直到执行完毕。

> 在异步编程里面如何应该关心的大概有三个问题：
1. 如何生成这个数组（序列）
2. 写执行器，如何有序的执行完这个数组，在一个操作完成之后通知下一个操作开始执行
3. 看似上面已经完成了所有问题，但还有一个隐藏问题需要解决。在已知的任何事情从宏观的角度来讲，都是一个序列，从前到后的推移，犹如时间的推移。但是从微观的角度，必定会有事件的分支和合流（不知道这样说会不会被打）。如何处理这一部分的逻辑也是重中之重。

对第一个问题，我们直接用数组，可以列为函数的参数，可以像Promise一样链式调用最后组成，奥还有callback。

到第二个问题，我们很容易联想到runSequence这种函数名，于是雏形就有了

```js
`function runSequence(...seqs) { }`
```

拿到`seqs`就是干，一个一个执行呗，重要的就是如何一个一个执行。从这一点来看的化，每个异步操作都会有callback，只是callback的时机和写法不一样，或优或劣，从callback里调用下一次操作。我们可以把callback想成next借用koa的思想把序列执行完。这个要求每个操作都会调用并且只调用一次next，以最普通callback函数为例。

```js
function runSequence(...seqs) {

    let i = 0;

    const next = (index, ...args) => {

        if (i !== index) {

            console.warn('next call twice');

            return;

        }

        let arg, callback;

        if (typeof args[args.length - 1] === 'function') {

            arg = args.slice(0, args.length - 1);

            callback = args[args.length - 1];

        } else {

            arg = args;

            callback = null;

        }

        const seq = seqs[i++];

        if (!seq) {

            callback && callback(null, ...arg);

            return;

        }

        try {

            seq.call(null, ...arg, ((i, err, ...res) => {

                if (err) {

                    callback && callback(err);

                } else {

                    next.call(null, i, ...res, callback);

                }

            }).bind(null, i));

        } catch (err) {

            callback && callback(err);

        }

    }

    return (...args) => {

        next.call(null, 0, ...args);

    }

}

```

大功告成，我们来写一些简单的例子测试一下

```js
const res = {

    a: 'b',

    b: 'c',

}



function fetch(key, next) {

    console.log(`fetch ${key} start`, Date.now() | 0);

    if (!key) {

        next('err key');

    }

    setTimeout(() => {

        console.log(`fetch ${key} end`, Date.now() | 0);

        if (!res[key]) {

            next(`${key} not found`);

        } else {

            next(null, res[key]);

        }

    }, 1000);

}



function runError(next) {

    throw new Error('erred');

    next();

}

runSequence(fetch, fetch, (data, next) => {

    next(null, data + '1');

    next(null, data + '2');

}, (data, next) => {

    console.log(data);

    next(null, data + '3');

}, runError)('a', (err, ...data) => {

    console.log(err, ...data, 'finish all');

});


```

log如下

```js
// fetch a start 5682

// fetch a end 6686

// fetch b start 6687

// fetch b start 6687

// c1

// Error: erred "finish all"

// warn: next call twice
```

看上去没毛病了。我们可以来解决第三个问题，还是拿来主义，借鉴`Promise.all`和`Promise.race`。我们也可以给callback写一个对应的all和race。直接贴代码

总的来说，任何方案，只要是有规范，按规范走就能很好的优化这个流程。比如最后一个函数是callback，callback最后，第一个参数是err等等。使用Promise也有一定遵循的，如果Promise.resolve(123, 345);，后面的参数就会被忽略了，如果使用了这一点runSequence也能优化很多。runSequence.all数组里只能用函数bind的形式来写，就很丑，就可以用thunkify来优化。

[原文地址](https://mp.weixin.qq.com/s/5swBKkITeG8yf9hZJbGfWg)