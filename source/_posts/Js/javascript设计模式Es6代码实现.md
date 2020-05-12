---
title: javascript设计模式Es6代码实现
tags: ['设计模式','ES6']
categories: 'Js'
date: 2020-05-12 13:20:35
---

[原文地址](https://juejin.im/post/5e021eb96fb9a01628014095)

# 设计模式简介

`设计模式`代表了最佳的实践，通常被有经验的面向对象的软件开发人员所采用。`设计模式`是软件开发人员在软件开发过程中面临的一般问题的解决方案。这些解决方案是众多软件开发人员经过相当长的一段时间的试验和错误总结出来的。

`设计模式`是一套被反复使用的、多数人知晓的、经过分类编目的、代码设计经验的总结。使用`设计模式`是为了重用代码、让代码更容易被他人理解、保证代码可靠性。 毫无疑问，`设计模式`于己于他人于系统都是多赢的，`设计模式`使代码编制真正工程化，`设计模式`是软件工程的基石，如同大厦的一块块砖石一样。

## 设计模式规则（SOLID）

**S – Single Responsibility Principle 单一职责原则**

* 一个程序只做好一件事
* 如果功能过于复杂就拆分开，每个部分保持独立


**O – OpenClosed Principle 开放/封闭原则**

* 对扩展开放，对修改封闭
* 增加需求时，扩展新代码，而非修改已有代码


**L – Liskov Substitution Principle 里氏替换原则**

* 子类能覆盖父类
* 父类能出现的地方子类就能出现


**I – Interface Segregation Principle 接口隔离原则**

* 保持接口的单一独立
* 类似单一职责原则，这里更关注接口


**D – Dependency Inversion Principle 依赖倒转原则**

* 面向接口编程，依赖于抽象而不依赖于具体
* 使用方只关注接口而不关注具体类的实现


再举个栗子：[此例来源-守候-改善代码的各方面问题](https://juejin.im/post/5adc8e18518825672b0352a8#comment)

```js
//checkType('165226226326','mobile')
//result：false
let checkType=function(str, type) {
    switch (type) {
        case 'email':
            return /^[\w-]+(\.[\w-]+)*@[\w-]+(\.[\w-]+)+$/.test(str)
        case 'mobile':
            return /^1[3|4|5|7|8][0-9]{9}$/.test(str);
        case 'tel':
            return /^(0\d{2,3}-\d{7,8})(-\d{1,4})?$/.test(str);
        default:
            return true;
    }
}
```

有以下两个问题：

1. 如果想添加其他规则就得在函数里面增加 case 。添加一个规则就修改一次！这样违反了开放-封闭原则（对扩展开放，对修改关闭）。而且这样也会导致整个 API 变得臃肿，难维护。
2. 比如A页面需要添加一个金额的校验，B页面需要一个日期的校验，但是金额的校验只在A页面需要，日期的校验只在B页面需要。如果一直添加 case 。就是导致A页面把只在B页面需要的校验规则也添加进去，造成不必要的开销。B页面也同理。

建议的方式是给这个 API 增加一个扩展的接口:

```js
let checkType=(function(){
    let rules={
        email(str){
            return /^[\w-]+(\.[\w-]+)*@[\w-]+(\.[\w-]+)+$/.test(str);
        },
        mobile(str){
            return /^1[3|4|5|7|8][0-9]{9}$/.test(str);
        }
    };
    //暴露接口
    return {
        //校验
        check(type,str){
            return rules[type]?rules[type](str):false;
        },
        //添加规则
        addRule(type,fn){
            rules[type]=fn;
        }
    }
})();

//调用方式
//使用mobile校验规则
console.log(checkType.check('mobile','188170239'));
//添加金额校验规则
checkType.addRule('money',function (str) {
    return /^[0-9]+(.[0-9]{2})?$/.test(str)
});
//使用金额校验规则
console.log(checkType.check('18.36','money'));
```
此例更详细内容请查看-> [守候i-重构-改善代码的各方面问题](https://juejin.im/post/5adc8e18518825672b0352a8#comment)

# 设计模式分类（23种）

## 目录

> 创建型
* 单例模式
* 原型模式
* 工厂模式
* 抽象工厂模式
* 建造者模式

> 结构型
* 适配器模式
* 装饰器模式
* 代理模式
* 外观模式
* 桥接模式
* 组合模式
* 享元模式

> 行为型
* 观察者模式
* 迭代器模式
* 策略模式
* 模板方法模式
* 职责链接模式
* 命令模式
* 备忘录模式
* 状态模式
* 访问者模式
* 中介者模式
* 解释器模式

## 工厂模式

### 定义

工厂模式定义到一个用于创建对象的接口，这个接口由子类决定实例化哪一类。该模式使一个类的实例化延迟到了子类,而子类可以重写接口方法以便创建的时候指定自己的对象类型

### code

```js
class Product{
  constructor(name){
    this.name = name
  }
  init(){
    console.log('init',this.name)
  }
}
class Factory {
  create(name){
    return new Product(name)
  }
}

let factory = new Factory()
let p = factory.create('hello world')
p.init()
```

### 使用场景

1. 如果你不想让某个子系统与较大的那个对象之间形成强耦合，而是想运行时从许多子系统中进行挑选的话，那么工厂模式是一个理想的选择
2. 将new操作简单封装，遇到new的时候就应该考虑是否用工厂模式；
3. 需要依赖具体环境创建不同实例，这些实例都有相同的行为,这时候我们可以使用工厂模式，简化实现的过程，同时也可以减少每种对象所需的代码量，有利于消除对象间的耦合，提供更大的灵活性

### 优点

* 创建对象的过程可能很复杂，但我们只需要关心创建结果。
* 构造函数和创建者分离, 符合“开闭原则”
* 一个调用者想创建一个对象，只要知道其名称就可以了。
* 扩展性高，如果想增加一个产品，只要扩展一个工厂类就可以。

### 缺点

* 添加新产品时，需要编写新的具体产品类,一定程度上增加了系统的复杂度
* 考虑到系统的可扩展性，需要引入抽象层，在客户端代码中均使用抽象层进行定义，增加了系统的抽象性和理解难度

### 什么时候不用

当被应用到错误的问题类型上时,这一模式会给应用程序引入大量不必要的复杂性.除非为创建对象提供一个接口是我们编写的库或者框架的一个设计上目标,否则我会建议使用明确的构造器,以避免不必要的开销。

由于对象的创建过程被高效的抽象在一个接口后面的事实,这也会给依赖于这个过程可能会有多复杂的单元测试带来问题。

### 例子

* 曾经我们熟悉的JQuery的$()就是一个工厂函数，它根据传入参数的不同创建元素或者去寻找上下文中的元素，创建成相应的jQuery对象

```js
class jQuery {
    constructor(selector) {
        super(selector)
    }
    add() {
        
    }
  // 此处省略若干API
}

window.$ = function(selector) {
    return new jQuery(selector)
}
```

* vue 的异步组件
在大型应用中，我们可能需要将应用分割成小一些的代码块，并且只在需要的时候才从服务器加载一个模块。为了简化，Vue 允许你以一个工厂函数的方式定义你的组件，这个工厂函数会异步解析你的组件定义。Vue 只有在这个组件需要被渲染的时候才会触发该工厂函数，且会把结果缓存起来供未来重渲染。例如：

```js
Vue.component('async-component',function(resolve,reject){
  setTimeout(funciton(){
    resolve({
      templete: '<div>I am async</div>'
    })
  },1000)
})
```

## 单例模式

### 定义

一个类只有一个实例，比提供一个访问它的全局访问点

### code

```js
 class LoginForm {
    constructor() {
        this.state = 'hide'
    }
    show() {
        if (this.state === 'show') {
            alert('已经显示')
            return
        }
        this.state = 'show'
        console.log('登录框显示成功')
    }
    hide() {
        if (this.state === 'hide') {
            alert('已经隐藏')
            return
        }
        this.state = 'hide'
        console.log('登录框隐藏成功')
    }
 }
 LoginForm.getInstance = (function () {
     let instance
     return function () {
        if (!instance) {
            instance = new LoginForm()
        }
        return instance
     }
 })()

let obj1 = LoginForm.getInstance()
obj1.show()

let obj2 = LoginForm.getInstance()
obj2.hide()

console.log(obj1 === obj2)

```

### 优点

* 划分命名空间，减少全局变量
* 增强模块性，把自己的代码组织在一个全局变量名下，放在单一位置，便于维护
* 且只会实例化一次。简化了代码的调试和维护

### 缺点

* 由于单例模式提供的是一种单点访问，所以它有可能导致模块间的强耦合 从而不利于单元测试。
* 无法单独测试一个调用了来自单例的方法的类，而只能把它与那个单例作为一个单元一起测试。

### 场景例子

定义命名空间和实现分支型方法
登录框
vuex 和 redux中的store

## 适配器模式

### 定义

将一个类接口转化为另一个接口，以满足用户需求，使类之间接口不兼容问题通过适配器得以解决

### code

```js
class Plug {
  getName() {
    return 'iphone充电头';
  }
}

class Target {
  constructor() {
    this.plug = new Plug();
  }
  getName() {
    return this.plug.getName() + ' 适配器Type-c充电头';
  }
}

let target = new Target();
target.getName(); // iphone充电头 适配器转Type-c充电头
```

### 优点

* 可以让两个没有关系的类连接起来一起运行
* 提高了类的复用
* 适配对象，适配库，适配数据

### 缺点

* 额外对象的创建，非直接调用，存在一定的开销（且不像代理模式在某些功能点上可实现性能优化)
* 如果没必要使用适配器模式的话，可以考虑重构，如果使用的话，尽量把文档完善

### 场景

* 整合第三方SDK
* 封装旧接口

```js
// 自己封装的ajax， 使用方式如下
ajax({
    url: '/getData',
    type: 'Post',
    dataType: 'json',
    data: {
        test: 111
    }
}).done(function() {})
// 因为历史原因，代码中全都是：
// $.ajax({....})

// 做一层适配器
var $ = {
    ajax: function (options) {
        return ajax(options)
    }
}
```

* vue的computed

```html
<template>
    <div id="example">
        <p>Original message: "{{ message }}"</p>  <!-- Hello -->
        <p>Computed reversed message: "{{ reversedMessage }}"</p>  <!-- olleH -->
    </div>
</template>
<script type='text/javascript'>
    export default {
        name: 'demo',
        data() {
            return {
                message: 'Hello'
            }
        },
        computed: {
            reversedMessage: function() {
                return this.message.split('').reverse().join('')
            }
        }
    }
</script>
```

**原有data 中的数据不满足当前的要求，通过计算属性的规则来适配成我们需要的格式，对原有数据并没有改变，只改变了原有数据的表现形式**

### 不同点

适配器模式与代理模式很相似

* 适配器模式：提供一个不同的接口（如不同版本的插头）
* 代理模式：提供一个一模一样的接口

## 装饰者模式

### 定义

动态地给某个对象添加一些额外的职责，，是一种实现继承的替代方案
在不改变原对象的基础上，通过对其进行包装扩展，使原有对象可以满足用户的更复杂需求，而不会影响从这个类中派生的其他对象

### code

```js
class Cellphone {
    create() {
        console.log('生成一个手机')
    }
}
class Decorator {
    constructor(cellphone) {
        this.cellphone = cellphone
    }
    create() {
        this.cellphone.create()
        this.createShell(cellphone)
    }
    createShell() {
        console.log('生成手机壳')
    }
}
// 测试代码
let cellphone = new Cellphone()
cellphone.create()

console.log('------------')
let dec = new Decorator(cellphone)
dec.create()
```

### 场景例子

* 比如现在有4 种型号的自行车，我们为每种自行车都定义了一个单独的类。现在要给每种自行车都装上前灯、尾灯和铃铛这3 种配件。如果使用继承的方式来给每种自行车创建子类，则需要 4×3 = 12 个子类。但是如果把前灯、尾灯、铃铛这些对象动态组合到自行车上面，则只需要额外增加3个类
* ES6 [Decorator](http://es6.ruanyifeng.com/#docs/decorator) 阮一峰
* core-decorators

### 优点

* 装饰类和被装饰类都只关心自身的核心业务，实现了解耦。
* 方便动态的扩展功能，且提供了比继承更多的灵活性。

### 缺点

* 多层装饰比较复杂。
* 常常会引入许多小对象，看起来比较相似，实际功能大相径庭，从而使得我们的应用程序架构变得复杂起来

## 代理模式

### 定义
是为一个对象提供一个代用品或占位符，以便控制对它的访问

> 假设当A 在心情好的时候收到花，小明表白成功的几率有60%，而当A 在心情差的时候收到花，小明表白的成功率无限趋近于0。
小明跟A 刚刚认识两天，还无法辨别A 什么时候心情好。如果不合时宜地把花送给A，花被直接扔掉的可能性很大，这束花可是小明吃了7 天泡面换来的。
但是A 的朋友B 却很了解A，所以小明只管把花交给B，B 会监听A 的心情变化，然后选择A 心情好的时候把花转交给A，代码如下：

```js
let Flower = function(){}
let xiaoming = {
  sendFlower: function(target){
    let flower = new Flower()
    target.receviceFlower()
  }
}
let B = {
  receviceFlower:function(flower){
    A.listenGoodMood(function(){
      A.receviceFlower(flower)
    })
  }
}
let A = {
  receviceFlower:function(flower){
    console.log('收到花'+ flower)
  }
  listenGoodMood:function(fn){
    setTimeout(function(){
      fn()
    },1000)
  }
}
xiaoming.sendFlower(B)
```

### 场景例子

* HTML元 素事件代理

```html
<ul id="ul">
  <li>1</li>
  <li>2</li>
  <li>3</li>
</ul>
<script>
  let ul = document.querySelector('#ul');
  ul.addEventListener('click', event => {
    console.log(event.target);
  });
</script>
```

* ES6 的 proxy 阮一峰Proxy
* jQuery.proxy()方法

### 优点

* 代理模式能将代理对象与被调用对象分离，降低了系统的耦合度。代理模式在客户端和目标对象之间起到一个中介作用，这样可以起到保护目标对象的作用
* 代理对象可以扩展目标对象的功能；通过修改代理对象就可以了，符合开闭原则；

### 缺点

* 处理请求速度可能有差别，非直接访问存在开销

### 不同点

> 装饰者模式实现上和代理模式类似
装饰者模式： 扩展功能，原有功能不变且可直接使用
代理模式： 显示原有功能，但是经过限制之后的

## 外观模式

### 定义

为子系统的一组接口提供一个一致的界面，定义了一个高层接口，这个接口使子系统更加容易使用

### code

1. 兼容浏览器事件绑定

```js
let addMyEvent = function (el, ev, fn) {
    if (el.addEventListener) {
        el.addEventListener(ev, fn, false)
    } else if (el.attachEvent) {
        el.attachEvent('on' + ev, fn)
    } else {
        el['on' + ev] = fn
    }
}; 
```

2. 封装接口

```js
let myEvent = {
    // ...
    stop: e => {
        e.stopPropagation();
        e.preventDefault();
    }
};
```
斜挎单肩包，透气跑鞋，运动袜，T恤，运动短裤

### 场景

* 设计初期，应该要有意识地将不同的两个层分离，比如经典的三层结构，在数据访问层和业务逻辑层、业务逻辑层和表示层之间建立外观Facade
* 在开发阶段，子系统往往因为不断的重构演化而变得越来越复杂，增加外观Facade可以提供一个简单的接口，减少他们之间的依赖。
* 在维护一个遗留的大型系统时，可能这个系统已经很难维护了，这时候使用外观Facade也是非常合适的，为系统开发一个外观Facade类，为设计粗糙和高度复杂的遗留代码提供比较清晰的接口，让新系统和Facade对象交互，Facade与遗留代码交互所有的复杂工作。

### 优点

* 减少系统的相互依赖
* 提高了灵活性
* 提高了安全性

### 缺点

不符合开闭原则，如果要改东西很麻烦，继承重写都不合适。

## 观察者模式


### 定义

一种一对多的关系，让多个观察者对象同时监听某一个主题对象，这个主题对象的状态发生变化时就会通知所有的观察者对象，使它们能够自动更新自己，当一个对象的改变需要同时改变其它对象，并且它不知道具体有多少对象需要改变的时候，就应该考虑使用观察者模式。

* 发布 & 订阅
* 一对多

### code

```js
// 主题 保存状态，状态变化之后触发所有观察者对象
class Subject {
  constructor() {
    this.state = 0
    this.observers = []
  }
  getState() {
    return this.state
  }
  setState(state) {
    this.state = state
    this.notifyAllObservers()
  }
  notifyAllObservers() {
    this.observers.forEach(observer => {
      observer.update()
    })
  }
  attach(observer) {
    this.observers.push(observer)
  }
}

// 观察者
class Observer {
  constructor(name, subject) {
    this.name = name
    this.subject = subject
    this.subject.attach(this)
  }
  update() {
    console.log(`${this.name} update, state: ${this.subject.getState()}`)
  }
}

// 测试
let s = new Subject()
let o1 = new Observer('o1', s)
let o2 = new Observer('02', s)

s.setState(12)
```

### 场景

* DOM事件

```js
document.body.addEventListener('click', function() {
    console.log('hello world!');
});
document.body.click()
```

* Vue响应式

### 优点

* 支持简单的广播通信，自动通知所有已经订阅过的对象
* 目标对象与观察者之间的抽象耦合关系能单独扩展以及重用
* 增加了灵活性
* 观察者模式所做的工作就是在解耦，让耦合的双方都依赖于抽象，而不是依赖于具体。从而使得各自的变化都不会影响到另一边的变化。

### 缺点

过度使用会导致对象与对象之间的联系弱化，会导致程序难以跟踪维护和理解

## 状态模式

### 定义

允许一个对象在其内部状态改变的时候改变它的行为，对象看起来似乎修改了它的类

### code

```js
class State {
  constructor(state){
    this.state = state
  }
  handle(context){
    console.log(`this is ${context} light`)
  }
}

class Context{
  constructor(){
    this.state = null
  }
  getState(){
    return this.state
  }
  setState(state){
    this.state = state
  }
}

let context = new Context()
let weak = new State('Weak')
let strong = new State('Strong')
let off = new State('off')

// 弱光
weak.handle(context)
console.log(context.getState())

// 强光
strong.handle(context)
console.log(context.getState())

// 关闭
off.handle(context)
console.log(context.getState())
```

### 场景

* 一个对象的行为取决于它的状态，并且它必须在运行时刻根据状态改变它的行为
* 一个操作中含有大量的分支语句，而且这些分支语句依赖于该对象的状态

### 优点

* 定义了状态与行为之间的关系，封装在一个类里，更直观清晰，增改方便
* 状态与状态间，行为与行为间彼此独立互不干扰
* 用对象代替字符串来记录当前状态，使得状态的切换更加一目了然

### 缺点

* 会在系统中定义许多状态类
* 逻辑分散
