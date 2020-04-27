---
title: call stack
date: 2018-11-04 23:12:27
tags: "Js"
categories: "Js"
---

# Call Stack(调用堆栈)

MDN(解释call Stack)

-----------

> A **call stack** is a mechanism for an interpreter (like the JavaScript interpreter in a web browser) to keep track of its place in a script that calls multiple functions — what function is currently being run, what functions are called from within that function and should be called next, etc.(调用堆栈是一种机制，用于解释器（如Web浏览器中的JavaScript解释器）跟踪其在调用多个函数的脚本中的位置——当前正在运行什么函数、在该函数中调用什么函数以及接下来应该调用什么函数，等等。)

* When a script calls a function, the interpreter adds it to the call stack and then starts carrying out the function.
* Any functions that are called by that function are added to the call stack further up, and run where their calls are reached.
* When the current function is finished, the interpreter takes it off the stack and resumes execution where it left off in the last code listing.
* If the stack takes up more space than it had assigned to it, it results in a "stack overflow" error.


## 调用栈

调用栈是一种数据结构。他记录了我们在程序中的位置；如果我们调用一个函数时，他就会将这个函数推入栈顶，当执行完这个函数后，就会从栈顶弹出，这就是调用栈所做的事情；

```js
function multiply(x,y){
    return x * y;
}
function printSquare(x){
    var s = multiply(x,x);
    console.log(s)
}
printSquare(5);

```

每一个进入调用栈的称为`调用帧`。
<!-- more -->
“堆栈溢出”，当你达到调用栈的最大的大小的时候就会发生这种情况；而且这相当容易发生，特别是在你写递归的时候却没有全方位的测试它。我们来看看下面的代码：

```js
function foo(){
    foo();
}

foo()
```

## 什么是执行上下文？

简而言之，执行上下文就是评估和执行JavaScript代码环境的抽象概念。每当JavaScript代码在运行时，他都是在执行上下文中运行的；

### 执行上下文的类型

* 全局执行上下文  这是默认或者说基础的上下文，任何不在函数内部的代码都在全局上下文中。它会执行两件事：创建一个全局的 window 对象（浏览器的情况下），并且设置 this 的值等于这个全局对象。**一个程序中只会有一个全局执行上下文**。

* 函数执行上下文  每当一个函数被调用时, 都会为该函数创建一个新的上下文。每个函数都有它自己的执行上下文，不过是在函数被调用时创建的。函数上下文可以有任意多个。每当一个新的执行上下文被创建，它会按定义的顺序（将在后文讨论）执行一系列步骤

* eval函数执行上下文 执行在 eval 函数内部的代码也会有它属于自己的执行上下文，但由于 JavaScript 开发者并不经常使用 eval，所以在这里我不会讨论它

### 执行栈

执行栈，也就是在其它编程语言中所说的“调用栈”，是**一种拥有 LIFO（后进先出）数据结构**的栈，被用来存储代码运行时创建的所有执行上下文
。
当 JavaScript 引擎第一次遇到你的脚本时，它会创建一个全局的执行上下文并且压入当前执行栈。每当引擎遇到一个函数调用，它会为该函数创建一个新的执行上下文并压入栈的顶部。
引擎会执行那些执行上下文位于栈顶的函数。当该函数执行结束时，执行上下文从栈中弹出，控制流程到达当前栈中的下一个上下文。

### 怎么创建执行上下文？

>在 JavaScript 代码执行前，执行上下文将经历创建阶段。在创建阶段会发生三件事：

1.this 值的决定，即我们所熟知的 This 绑定。
2.创建词法环境组件。
3.创建变量环境组件。