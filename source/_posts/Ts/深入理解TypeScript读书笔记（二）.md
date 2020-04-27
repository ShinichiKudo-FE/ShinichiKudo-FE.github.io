---
title: <深入理解TypeScript读书笔记>（一）
date: 2020-4-19 16:18:50
tags: "Ts"
categories: "Js"
---
# TypeScript类型系统

## @types

### 全局@types
@types 支持全局和模块类型定义

```js
npm install @type/jquery --save-dev
```
默认情况下，TypeScript 会自动包含支持全局使用的任何声明定义

### 局部@types
安装完成后不需要做特别配置，只要想使用模块一样使用它一样

```js
import * as $ from 'jquery'
```

可以在`typeconfig.json`中可以配置有意义的类型

```js
{
    compilerOptions{
        "type" :[
            "jquery"
        ]
    }
}
```

## 声明文件

可以使用`declear`关键字来告诉typescript，你试图表诉一个其他地方已经存在的代码

```ts
foo = 123
declear var foo:any
foo = 123
```

在实际项目可以新建一个`global.d.ts`文件来把这些声明放到里面

## 接口

接口运行时的影响为0.有多种方式可以声明变量的结构

```js
//等效声明
//内联注解
declear const myPoint = {x:number,y:number}
//接口实现  
interface myPoint{
    x: number,
    y: number
}
```

### 类来实现接口

```js
interface myPoint{
    x: string,
    y: string
}

class point implements myPoint{
    x: string,
    y: string
}
```

**注意：并非每个接口都容易实现**
```js
interface Crazy{
    new() {
        value: number
    }
}

class CrazyClass implements Crazy{
    constructor(){
        return {
            value: 123
        }
    }
}

const crazy = new CrazyClass()
```

## 枚举

枚举是收集有关连变量的一种方式, 作用就是管理常量，让常量更规范统一

### 数字枚举

```js
enum CradSuit{
    Clubs,
    Diamonds,
    Hearts,
    Spades
}
let clubs = CradSuot.Clubs
```
### 字符串枚举

```js
enum CradSuit{
    Clubs,
    Diamonds,
    Hearts,
    Spades
}
console.log(CardSuit.Clubs[0]) // 'Clubs'
console.log(CardSuit.['Diamonds']) // 0
```
改变与枚举关联的数字

```js
enum CradSuit{
    Clubs , // 0
    Diamonds, // 1
    Hearts, // 2
    Spades // 3
}
enum CradSuit{
    Clubs  = 3, // 3
    Diamonds, // 4
    Hearts, // 5
    Spades // 6

```
   
### 常量枚举

```js
enum Fruit {
    Apple,
    banner,
    orange
}

console.log(Fruit.orange) // 2
```

#### 静态方法的枚举

```js
enum Weekday{
    Monday,
    Tuesday,
    Wednesday,
    Thursday,
    Friday,
    Saturday
    Sunday
}
namesapce Weekday{
    export function isBusinessDay(weekday: Weekday){
        switch(weekday){
            case Weekday.Saturday:
        case Weekday.Sunday:
            return false;
        default:
            return true;
        }
    }
}
const mon = Weekday.Monday;
const sun = Weekday.Sunday;

console.log(Weekday.isBusinessDay(mon)); // true
console.log(Weekday.isBusinessDay(sun));
```

## lib.d.ts

安装typescript的时候会有附带一个`lib.d.ts`的文件，里面的内容定义了(window,document,math，data,string)的变量声明

## 函数

可组合系统的核心构建块

### 返回函数类型注解
```js
interface Foo {
    foo: string
}

function foo(sameple: foo) :foo{
    console.log(foo)
}
```

### 可选参数及默认参数值属性
```js
function person(speak:string ="hello world",age?:number){
    console.log(speak,age)
}
person('你好')
person(...12)
```

### 函数重载

```js
function addOne(a:number,y?:number,z?:number,x?:number){
    if(x === undefined && y === undefined && z === undefined){
        x = y = z = a
    }else if( y === undefined && z === undefined ){
        z = a 
        x = y
    }
    return {
        top: a,
        left: x,
        right: y,
        bottom: z
    }
}
addOne(1)
addOne(1,1,1,1)
```

### 函数声明

```js
type londHand = {
    (a:number) : number
}
type ShortHand = (a:number) =>{number}
```

## 可调用的

```js
interface Complex {
    (foo:string,age?:number,...other,boolean[]):number
}
```

## 类型断言

TypeScript 允许你覆盖它的推断，并且能以你任何你想要的方式分析它，这种机制被称为「类型断言」

`as foo 与 <foo>`

`<foo>`可能会与jsx中冲突，推荐使用`as foo`

### 双重断言

```js
function handle(event:Event){
    console.log(element = (event as any) as HTMLElement)
}
```

## Freshness

为了能让检查对象字面量类型更容易，提供Freshness更严格的类型检查

## 类型保护

你可以使用更小范围的对象类型

`typeof` `instanceof`

## 字面量类型

你可以使用一个准备的类型来定义一个对象

## readOnly

如字面意思一样，是只读的，不能随便更改的，是一种更安全的方式

```js
type Foo = {
  readonly bar: number;
  readonly bas: number;
};

// 初始化
const foo: Foo = { bar: 123, bas: 456 };

// 不能被改变
foo.bar = 456; // Error: foo.bar 为仅读属性
```

### 与const不同

* const 用于变量，变量不能重新赋给其他任何事物
* readOnly 用于属性，用于别名可以修改属性

## 泛型
他的用途就是在你不确定要穿的值的属性时，那么就可以使用泛型去确定
在成员之间提供有意义的约束，这些成员可以是：

* 类的实例成员
* 类的方法
* 函数参数
* 函数返回值

```js
class Queue<T>{
    private data: T[] = []
    push = (item:<T>) => {this.data.push(item)}
    pop = ():T | undefiend => {this.data.shift()}
}

const queue = new Queue<number>()
queue.push(1)
queue.shift('1') //  error '1' is string
```
你可以随意调用泛型参数，当你使用简单的泛型时，泛型常用 T、U、V 表示

## 类型推断

TypeScript 能根据一些简单的规则推断（检查）变量的类型，你可以通过实践，很快的了解它们。

定义变量，函数返回类型，赋值，解构，结构化等都可以推断出类型

## 类型兼容性
用于确定一个类型是否能赋值给其他类型

协变
逆变，
双向协变，
不变

## never

`never` 类型是 TypeScript 中的底层类型。它自然被分配的一些例子：

一个从来不会有返回值的函数（如：如果函数内含有 `while(true) {}`）；
一个总是会抛出错误的函数（如：`function foo() { throw new Error('Not Implemented')` }，foo 的返回类型是 never）；

但是，never 类型仅能被赋值给另外一个 never

### 用例：详细的检查

```js
function foo(x:string | number):boolean {
    if (typeof x === 'string') {
        return true;
    } else if (typeof x === 'number') {
        return false;
    }
    // 如果不是一个 never 类型，这会报错：
    // - 不是所有条件都有返回值 （严格模式下）
    // - 或者检查到无法访问的代码
    // 但是由于 TypeScript 理解 `fail` 函数返回为 `never` 类型
    // 它可以让你调用它，因为你可能会在运行时用它来做安全或者详细的检查。
    return fail('Unexhaustive');
} 
function fail(message: string): never {
  throw new Error(message);
}
```

### never与void的区别

void 表示没有任何类型，never 表示永远不存在的值的类型。

## 辨析联合类型

当类中含有字面量成员时，我们可以用该类的属性来辨析联合类型。
```js
interface Square {
  kind: 'square';
  size: number;
}

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

type Shape = Square | Rectangle;
```

## 流动的类型

TypeScript 类型系统非常强大，它支持其他任何单一语言无法实现的类型流动和类型片段。
关键的动机：当你改变了其中一个时，其他相关的会自动更新，并且当有事情变糟糕时，你会得到一个友好的提示，就好像一个被精心设计过的约束系统。

### 复制类型和值
如果你想移动一个类，你可能会想要做以下事情：

```js
class Foo {}

const Bar = Foo;

let bar: Bar; // Error: 不能找到名称 'Bar'

//这会得到一个错误，因为 const 仅仅是复制了 Foo 到一个变量声明空间，因此你无法把 Bar 当作一个类型声明使用。正确的方式是使用 import 关键字，请注意，如果你在使用 namespace 或者 modules，使用 import 是你唯一能用的方式：
namespace importing {
  export class Foo {}
}

import Bar = importing.Foo;
let bar: Bar; // ok

//这个 import 技巧，仅适合于类型和变量。
```

### 捕获变量的类型

你可以通过 typeof 操作符在类型注解中使用变量    

## 异常处理

JavaScript 有一个 Error 类，用于处理异常。你可以通过 throw 关键字来抛出一个错误。然后通过 try/catch 块来捕获此错误：

```js
try{
    throw new Error('Something bad happened');
}catch(e){
    console.log(e)
}
```

除内置的 Error 类外，还有一些额外的内置错误，它们继承自 Error 类：`RangeError,ReferenceError,SyntaxError,TypeError,URIError`

## 混合

TypeScript (和 JavaScript) 类只能严格的单继承，因此你不能做：

```js
class User extends Tagged, Timestamped { // ERROR : 不能多重继承
  // ..
}
```
从可重用组件构建类的另一种方式是通过基类来构建它们，这种方式称为混合。

## 常见的typescript错误

[常见的typescript错误](https://jkchao.github.io/typescript-book-chinese/error/common.html#ts2304)

