---
title: JS高级之面试必须知道的几个点
date: 2018-04-16 16:27:55
tags: "Interview"
categories: "Interview"
---

# 函数的3种定义

## 函数声明

```javascript
//ES5
function getSum(){}
function (){}//匿名函数
//ES6
()=>{}//如果{}内容只有一行{}和return关键字可省,
```

## 函数表达式（字面量形式）

```javascript
//ES5
var sum=function getSum(){}
//ES6
let sum=()=>{}//如果{}内容只有一行{}和return关键字可省,
```

## 构造函数

```javascript
var sum=new GetSum(num1,num2)
```

## 三种方法的对比

1.函数声明有预解析,而且函数声明的优先级高于变量;
2.使用Function构造函数定义函数的方式是一个函数表达式,这种方式会导致解析两次代码，影响性能。第一次解析常规的JavaScript代码，第二次解析传入构造函数的字符串

## Es5函数中的4种调用

在ES5中函数内容的this指向和调用方法有关

### 函数调用模式

包括函数名()和匿名函数调用,`this指向window`

```javascript
 function getSum() {
    console.log(this) //window
 }
 getSum()

 (function() {
    console.log(this) //window
 })()

 var getSum=function() {
    console.log(this) //window
 }
```

<!-- more -->

### 方法调用

对象.方法名(),`this指向对象`

```javascript
var objList = {
   name: 'methods',
   getSum: function() {
     console.log(this) //objList对象
   }
}
objList.getSum()
```

### 构造器调用

new 构造函数名(),`this指向构造函数`

```javascript
function Person() {
  console.log(this); //指向构造函数Person
}
var personOne = new Person();
```

### 间接调用

利用call和apply来实现,this就是call和apply对应的第一个参数,如果不传值或者第一个值为null,`undefined时this指向window`

```javascript
function foo() {
   console.log(this);
}
foo.apply('我是apply改变的this值');//我是apply改变的this值
foo.call('我是call改变的this值');//我是call改变的this值
```

### Es6函数的调用

箭头函数不可以当作构造函数使用，也就是不能用new命令实例化一个对象，否则会抛出一个错误
箭头函数的this是和定义时有关和调用无关
调用就是函数调用模式

```javascript
(() => {
   console.log(this)//window
})()

let arrowFun = () => {
  console.log(this)//window
}
arrowFun()

let arrowObj = {
  arrFun: function() {
   (() => {
     console.log(this)//arrowObj
   })()
   }
 }
 arrowObj.arrFun();
```

## call,apply和bind

1.IE5之前不支持call和apply,bind是ES5出来的;
2.call和apply可以调用函数,改变this,实现继承和借用别的对象的方法;

### call和apply定义

调用方法,用一个对象替换掉另一个对象(this)
对象.call(新this对象,实参1,实参2,实参3.....) 参数列表
对象.apply(新this对象,[实参1,实参2,实参3.....]) 参数数组

### call和apply用法

1.间接调用函数,改变作用域的this值
2.劫持其他对象的方法

```javascript
var foo = {
  name:"张三",
  logName:function(){
    console.log(this.name);
  }
}
var bar={
  name:"李四"
};
foo.logName.call(bar);//李四
实质是call改变了foo的this指向为bar,并调用该函数
```
3.两个函数实现继承

```javascript
function Animal(name){
  this.name = name;
  this.showName = function(){
  console.log(this.name);
  }
}
function Cat(name){  
  Animal.call(this, name);  
} 
var cat = new Cat("Black Cat");
cat.showName(); //Black Cat
```

4.为类数组(arguments和nodeList)添加数组方法push,pop

```javascript
(function(){
  Array.prototype.push.call(arguments,'王五');
  console.log(arguments);//['张三','李四','王五']
})('张三','李四')
```

5.合并数组

```javascript
let arr1=[1,2,3]; 
let arr2=[4,5,6]; 
Array.prototype.push.apply(arr1,arr2); //将arr2合并到了arr1中
```

6.求数组最大值

```javascript
Math.max.apply(null,arr)
```

7.判断字符类型

```javascript
Object.prototype.toString.call({})
```

### bind

bind是function的一个函数扩展方法，bind以后代码重新绑定了func内部的this指向,不会调用方法,不兼容IE8

```javascript
var name = '李四'
 var foo = {
   name: "张三",
   logName: function(age) {
   console.log(this.name, age);
   }
 }
 var fooNew = foo.logName;
 var fooNewBind = foo.logName.bind(foo);
 fooNew(10)//李四,10
 fooNewBind(11)//张三,11  因为bind改变了fooNewBind里面的this指向
```

## 常见的四种设计模式

### 工厂模式

简单的工厂模式可以理解为解决多个相似的问题;

```javascript
function CreatePerson(name,age,sex) {
    var obj = new Object();
    obj.name = name;
    obj.age = age;
    obj.sex = sex;
    obj.sayName = function(){
        return this.name;
    }
    return obj;
}
var p1 = new CreatePerson("longen",'28','男');
var p2 = new CreatePerson("tugenhua",'27','女');
console.log(p1.name); // longen
console.log(p1.age);  // 28
console.log(p1.sex);  // 男
console.log(p1.sayName()); // longen

console.log(p2.name);  // tugenhua
console.log(p2.age);   // 27
console.log(p2.sex);   // 女
console.log(p2.sayName()); // tugenhua  
```

### 单例模式

只能被实例化(构造函数给实例添加属性与方法)一次

```javascript
// 单体模式
var Singleton = function(name){
    this.name = name;
};
Singleton.prototype.getName = function(){
    return this.name;
}
// 获取实例对象
var getInstance = (function() {
    var instance = null;
    return function(name) {
        if(!instance) {//相当于一个一次性阀门,只能实例化一次
            instance = new Singleton(name);
        }
        return instance;
    }
})();
// 测试单体模式的实例,所以a===b
var a = getInstance("aa");
var b = getInstance("bb");  
```

### 沙箱模式

将一些函数放到自执行函数里面,但要用闭包暴露接口,用变量接收暴露的接口,再调用里面的值,否则无法使用里面的值

```javascript
let sandboxModel=(function(){
    function sayName(){};
    function sayAge(){};
    return{
        sayName:sayName,
        sayAge:sayAge
    }
})()
```

### 发布者订阅模式

就例如如我们关注了某一个公众号,然后他对应的有新的消息就会给你推送,

```javascript
//发布者与订阅模式
    var shoeObj = {}; // 定义发布者
    shoeObj.list = []; // 缓存列表 存放订阅者回调函数

    // 增加订阅者
    shoeObj.listen = function(fn) {
        shoeObj.list.push(fn); // 订阅消息添加到缓存列表
    }

    // 发布消息
    shoeObj.trigger = function() {
            for (var i = 0, fn; fn = this.list[i++];) {
                fn.apply(this, arguments);//第一个参数只是改变fn的this,
            }
        }
     // 小红订阅如下消息
    shoeObj.listen(function(color, size) {
        console.log("颜色是：" + color);
        console.log("尺码是：" + size);
    });

    // 小花订阅如下消息
    shoeObj.listen(function(color, size) {
        console.log("再次打印颜色是：" + color);
        console.log("再次打印尺码是：" + size);
    });
    shoeObj.trigger("红色", 40);
    shoeObj.trigger("黑色", 42);  
```

代码实现逻辑是用数组存贮订阅者, 发布者回调函数里面通知的方式是遍历订阅者数组,并将发布者内容传入订阅者数组

更多设计模式请戳:[Javascript常用的设计模式详解](https://www.cnblogs.com/tugenhua0707/p/5198407.html)

## 原型链

### 定义

对象继承属性的一个链条

### 构造函数,实例与原型对象的关系

[构造函数,实例与原型对象的关系](https://sfault-image.b0.upaiyun.com/425/132/4251326904-5ad1d715e4939_articlex)

```javascript
var Person = function (name) { this.name = name; }//person是构造函数
var o3personTwo = new Person('personTwo')//personTwo是实例
```

[构造函数,实例与原型对象的关系例子](https://sfault-image.b0.upaiyun.com/187/926/1879268198-5ad1d87845b9d_articlex)

原型对象都有一个默认的constructor属性指向构造函数

## 创建实例的方法

1.字面量

```javascript
let obj={'name':'张三'}
```

2.Object构造函数创建

```javascript
let Obj=new Object()
Obj.name='张三'
```

3.使用工厂模式创建对象

```javascript
function createPerson(name){
 var o = new Object();
 o.name = name;
 };
 return o;
}
var person1 = createPerson('张三');
```

4.使用构造函数创建对象

```javascript
function Person(name){
 this.name = name;
}
var person1 = new Person('张三');
```

## new运算符

1.创了一个新对象;
2.this指向构造函数;
3.构造函数有返回,会替换new出来的对象,如果没有就是new出来的对象
4.手动封装一个new运算符

```javascript
var new2 = function (func) {
    var o = Object.create(func.prototype); 　　 //创建对象
    var k = func.call(o);　　　　　　　　　　　　　//改变this指向，把结果付给k
    if (typeof k === 'object') {　　　　　　　　　//判断k的类型是不是对象
        return k;　　　　　　　　　　　　　　　　　 //是，返回k
    } else {
        return o;　　　　　　　　　　　　　　　　　 //不是返回返回构造函数的执行结果
    }
}  
```

更多详情:[详谈JavaScript原型链](https://www.cnblogs.com/chengzp/p/prototype.html)

## 对象的原型链

[对象的原型链](https://sfault-image.b0.upaiyun.com/416/875/4168756668-5ad1deb66d66a_articlex)

## 继承的方式

JS是一门弱类型动态语言,封装和继承是他的两大特性

### 原型链继承

将父类的实例作为子类的原型
**代码实现**

定义父类:

```javascript
// 定义一个动物类
function Animal (name) {
  // 属性
  this.name = name || 'Animal';
  // 实例方法
  this.sleep = function(){
    console.log(this.name + '正在睡觉！');
  }
}
// 原型方法
Animal.prototype.eat = function(food) {
  console.log(this.name + '正在吃：' + food);
};
```

子类:

```javascript
function Cat(){
}
Cat.prototype = new Animal();
Cat.prototype.name = 'cat';

//　Test Code
var cat = new Cat();
console.log(cat.name);//cat
console.log(cat.eat('fish'));//cat正在吃：fish  undefined
console.log(cat.sleep());//cat正在睡觉！ undefined
console.log(cat instanceof Animal); //true
console.log(cat instanceof Cat); //true
```

**优缺点**
简单易于实现,但是要想为子类新增属性和方法，必须要在new Animal()这样的语句之后执行,无法实现多继承

### 构造继承

实质是利用call来改变Cat中的this指向
**代码实现**
子类:

```javascript
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}
```

**优缺点**
可以实现多继承,不能继承原型属性/方法

### 实例继承
为父类实例添加新特性，作为子类实例返回
**代码实现**
子类
```javascript
function Cat(name){
  var instance = new Animal();
  instance.name = name || 'Tom';
  return instance;
}
```
**优缺点**
不限制调用方式,但不能实现多继承

### 拷贝继承
将父类的属性和方法拷贝一份到子类中
子类:

```javascript
function Cat(name){
  var animal = new Animal();
  for(var p in animal){
    Cat.prototype[p] = animal[p];
  }
  Cat.prototype.name = name || 'Tom';
}
```
**优缺点**
支持多继承,但是效率低占用内存

### 组合继承
通过调用父类构造，继承父类的属性并保留传参的优点，然后通过将父类实例作为子类原型，实现函数复用
子类:
```javascript
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}
Cat.prototype = new Animal();
Cat.prototype.constructor = Cat;
```

###  寄生组合继承
```javascript
function Cat(name){
  Animal.call(this);
  this.name = name || 'Tom';
}
(function(){
  // 创建一个没有实例方法的类
  var Super = function(){};
  Super.prototype = Animal.prototype;
  //将实例作为子类的原型
  Cat.prototype = new Super();
})();
```

### ES6的extends继承
ES6 的继承机制是先创造父类的实例对象this（所以必须先调用super方法），然后再用子类的构造函数修改this,链接描述
```javascript
class ColorPoint extends Point {
  constructor(x, y, color) {
    super(x, y); // 调用父类的constructor(x, y)
    this.color = color;
  }

  toString() {
    return this.color + ' ' + super.toString(); // 调用父类的toString()
  }
}   
```

更多详情请戳:[JS继承的实现方式](https://www.cnblogs.com/humin/p/4556820.html#undefined)

[原文地址](https://segmentfault.com/a/1190000014405410?utm_source=channel-hottest#articleHeader16)