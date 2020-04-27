---
title: JavaScript中的“this”是什么？
date: 2018-08-05 23:19:50
tags: "This"
categories: "Js"
---
> 原文：[What is “this” in JavaScript?](https://medium.com/m/global-identity?redirectUrl=https%3A%2F%2Fblog.bitsrc.io%2Fwhat-is-this-in-javascript-3b03480514a7)

# 前言

如果你曾使用JavaScript库做过开发，那么你可能已经注意到一个名为 this的特定关键字。虽然 this在JavaScript中非常常见，但是完全理解this关键字的原理以及在代码中如何使用对相当一部分的开发者来说着实不易。在这篇文章中，我将帮你深入理解 this及其工作机制。
在开始之前，请确保已安装 Node。然后，打开命令行终端并运行 node 命令。

## 全局环境中的“this”

this的工作机制理解起来并不是很容易。我们通过将 this置于不同环境下，分别来理解 this是如何工作的。首先看一下 global环境。

在全局环境中， this相当于全局对象 global。

```js
this === global;
true;
```

但这只在 **node中才有效。如果我们将相同的代码放在js文件中运行，得到的输出为false**。

如果想测试效果，可以创建一个名为 index.js的文件，包含以下代码：

```js
console.log(this === global);
```

然后使用 node命令运行此文件：

```js
$ node index.js
false
```

原因是在JavaScript文件中， this等同于 `module.exports`而不是 global 。

## 函数中的“this”

函数中的 this值通常是由函数的调用方来定义。因此，每次执行函数，函数内的 this值可能都不一样。

在 index.js文件中，编写一个非常简单的函数，来检查 this是否等于global对象。

```js
function rajat() {
  console.log(this === global)
}
rajat()
```

<!-- more -->
如果我们使用 node运行此代码，得到的输出为 true 。如果在文件的顶部添加 "use strict"，并再次运行它，将会得到一个 false输出，因为现在的 this值为 undefined 。
为了进一步解释这一点，让我们创建一个简单的函数来定义超级英雄的真实姓名和英雄名字

```js
function Hero(heroName, realName) {
  this.realName = realName;
  this.heroName = heroName;
}
const superman= Hero("Superman", "Clark Kent");
console.log(superman);
```

请注意，这个函数并不是以`严格模式`编写的。 node运行此代码，并不会得到我们预期的“Superman”和“Clark Kent”，但它只会给我们一个 undefined 。

背后的原因是，**由于函数不是以严格模式编写的， this指向global对象**。
如果在`严格模式`下运行此代码，我们会收到报错，因为JavaScript不允许将属性 realName和 heroName赋给 undefined 。其实这是一件好事，因为它阻止我们创建全局变量。

另外，以大写形式书写函数名意味着我们需要将它视为构造函数，使用 new运算符来调用。用下面的代码替换上面代码段的最后两行

```js
onst superman = new Hero("Superman", "Clark Kent");
console.log(superman);
```

再次运行 node index.js命令，现在会得到你预期的输出。

## 构造函数中的“this”

JavaScript没有特定的构造函数。我们所能做的就是使用 new运算符将函数调用转换为构造函数调用，如上一节所示。
构造函数被调用时，会创建一个新对象并将其设置为函数的 this参数。构造函数会隐式的返回这个对象，除非我们明确的返回了另外一个对象。
在 hero函数内部添加下面的 return语句：

```js
return {
  heroName: "Batman",
  realName: "Bruce Wayne",
};
```

如果我们再次运行 node命令，我们会看到 上面的 return语句覆盖了构造函数调用。

return语句不会覆盖构造函数调用的唯一情形是，**return语句返回的不是一个对象**。在这种情况下，对象将包含原始值。

## 方法中的“this”

当函数作为对象的方法被调用时， this指向的是该对象，也称为函数调用的接收器(receiver)。
假设 hero对象有一个 dialogue方法 ，那么 dialogue中的 this值指向 hero本身。此时的 hero也被称为 dialogue方法调用的接收者。

```js
const hero = {
  heroName: "Batman",
  dialogue() {
    console.log(`I am ${this.heroName}!`);
  }
};
hero.dialogue();

```

这个例子非常简单，但在实际情况中，我们的方法很难跟踪接收器。在 index.js的末尾添加以下代码段。

```js
const saying = hero.dialogue;
saying();
```

如果我将 dialogue的引用存储在另一个变量中，并将该变量作为函数调用。 node运行代码 ， this将返回 undefined ，因为该方法已经丢失了接收器。 this此时指向 global ，而不是 hero 。

当我们将一个方法作为回调传递给另一个方法时，通常会丢失接收器。我们可以通过添加包装函数或使用 `bind`方法将 this与特定对象绑定来解决这个问题。

## call() 和 apply()

虽然函数的 this值是隐式设置的，我们在调用函数时也可以使用 call()和 apply()明确设置 this参数。

我们重构一下前面章节的代码片段，如下所示：

```js
function dialogue () {
  console.log (`I am ${this.heroName}`);
}
const hero = {
  heroName: 'Batman',
};

```

如果要将 hero对象作为 dialogue函数的接收器，我们可以这样使用 call()或 apply() ：

```js
dialogue.call(hero)
// or
dialogue.apply(hero)
```

如果你是在非严格模式下使用 call或 apply ，JavaScript引擎会忽略传递给 call或 apply的 null或 undefined （译者注：被替换为global）。这也是为什么建议始终以严格模式编写代码的原因之一。

## bind()

当我们将一个方法作为回调函数传递给另一个函数时，总是存在丢失方法的原有接收器的风险，使得 this参数指向全局对象。

`bind()`方法可以将 this参数固定的绑定到一个值上。下面的代码片段， bind会创建一个新的 dialogue函数，并将 this值设置为 hero 。（译者注：`bind()`方法会创建一个新函数，称为绑定函数-bound function-BF，当调用这个绑定函数时，绑定函数会以创建它时传入 `bind()`方法的第一个参数作为 this，传入 `bind()`方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。）

```js
const hero = {
  heroName: "Batman",
  dialogue() {
    console.log(`I am ${this.heroName}`);
  }
};
setTimeOut(hero.dialogue.bind(hero), 1000);

```

这样的话，即使使用 call或 apply方法也无法改变 this的值 。

## 箭头函数中的“this”

箭头函数内的 this与其他类型的JavaScript函数有很大的不同。An arrow function uses the this value from its enclosing execution context, since it does have one of its own.

箭头函数会**永久地捕获 this值，阻止 apply或 call后续更改它**。

为了解释箭头函数中的 this是如何工作的，我们来写一个箭头函数：

```js
const batman = this;
const bruce = () => {
  console.log(this === batman);
};
bruce(); //  true

```

这里，我们将 this值存储在变量中，然后将该值与箭头函数内的 this值进行比较。在终端中运行 node index.js，输出应该为 true 。

箭头函数内的 this值无法明确设置。此外，使用 call 、 apply或 bind等方法给 this传值，箭头函数会忽略。**箭头函数引用的是箭头函数在创建时设置的 this值**。（译者注：**箭头函数中没有this绑定，必须通过查找作用域链来决定它的值，如果箭头函数被非箭头函数包裹，那么this值由外围最近一层非箭头函数决定，否则为undefined。**）

箭头函数也不能用作构造函数。因此，我们也不能在箭头函数内给 this设置属性。

那么箭头函数对 this 可以做什么呢？

箭头函数 **可以使我们在回调函数中访问this** 。通过下面的 counter对象来看看如何做到的：

```js
const counter = {
  count: 0,
  increase() {
    setInterval(function() {
      console.log(++this.count);
    }, 1000);
  }
}
counter.increase();
```

使用 node index.js运行此代码，只会得到一个 NaN的列表。这是因为 this.count已经不是指向 counter对象了。它实际上指向的为 global对象。

如果想让计数器正常工作，可以使用箭头函数重写它。

回调函数使用 this与 increase方法绑定， counter现在可以正常工作了。

注意 ：不要将 `++this.count` 写成 this.count + 1。后者只会增加count的值一次，每次迭代都会返回相同的值。

## Class中的“this”

类是JavaScript应用程序中最重要的一部分。让我们看看类中的this有何不同。

类通常包含一个 constructor ， **this可以指向任何新创建的对象**。

不过在作为方法时，如果该方法作为普通函数被调用， this也可以指向任何其他值。与方法一样，类也可能失去对接收器的跟踪。

我们将之前创建的 Hero函数改造为类。该类包含一个构造函数和一个 dialogue()方法。最后，创建一个类的实例并调用 dialogue方法。

```js
class Hero {
  constructor(heroName) {
    this.heroName = heroName;
  }
  dialogue() {
    console.log(`I am ${this.heroName}`)
  }
}
const batman = new Hero("Batman");
batman.dialogue();
```

构造函数里的 this指向新创建的 类实例。当我们调用 `batman.dialogue()`时， dialogue()作为方法被调用， batman是它的接收器。

但是如果我们将 `dialogue()`方法的引用存储起来，并稍后将其作为函数调用，我们会丢失该方法的接收器，此时 this参数指向 `undefined`

```js
const say = batman.dialogue;
say();
```

出现错误的原因是JavaScript 类是隐式的运行在严格模式下的。我们是在没有任何自动绑定的情况下调用 say()函数的。要解决这个问题，我们需要手动使用 bind()将 dialogue()函数与 batman绑定在一起。

```js
const say = batman.dialogue.bind(batman);
say();
```

我们也可以在 构造函数方法中做这个绑定。

## 总结

我们需要在JavaScript中使用 this ，就像我们需要在英语中使用代词一样。以这两句话为例：

> * Rajat loves DC Comics.
> * Rajat also loves Marvel movies.

我们可以使用代词将这两个句子组合在一起，所以这两句话现在成了：

> * Rajat loves DC Comics, and he also loves Marvel Comics

这个简短的语法课完美地解释了 this在JavaScript中的重要性。就像代词 he将两个句子连接在一起一样， this可以作为再次引用相同内容的捷径。

[原文地址](https://juejin.im/post/5b6676e6f265da0fa00a3a12?utm_source=gold_browser_extension#comment)