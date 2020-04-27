---
title: 在17道题中理解javascript的类型转换
date: 2019-08-08 15:50:44
tags: "Js"
categories: "Js"
---

[原文地址](https://juejin.im/post/5d4999fff265da038f47f5c7)

类型转换是将值从一种类型转换为另一种类型的过程（比如字符串转数字，对象转布尔值等）。任何类型不论是原始类型还是对象类型都可以进行类型转换，JavaScript 的原始类型有：`number, string, boolean, null, undefined, Symbol`。

```js
true + false
12 / "6"
"number" + 15 + 3
15 + 3 + "number"
[1] > null
"foo" + + "bar"
"true" == true
false == "false"
null == ""
!!"false" == !!"true"
["x"] == "x"
[] + null + 1
[1,2,3] == [1,2,3]
{} + [] + {} + [1]
! + [] + [] + ![]
new Date(0) - 0
new Date(0) + 0
```

类似于上面的这些问题大概率也会在 JS 面试中被问到, 所以继续往下读。

## 隐式转换VS显示转换

类型转换可以分为**隐式类型转换**和**显式类型转换**。

当开发人员通过编写适当的代码(如Number(value))用于在类型之间进行转换时，就称为`显式类型强制转换(或强制类型转换)`。

然而 JavaScript 是弱类型语言，在某些操作下，值可以在两种类型之间自动的转换，这叫做`隐式类型转换`。在对不同类型的值使用运算符时通常会发生隐式类型转换。比如 1 == null, 2 / "5", null + new Date()。当值被 if 语句包裹时也有可能发生，比如 if(value) {} 会将 value 转换为 boolean类型。

`严格相等运算符`（===）不会触发类型隐式转换，所以它可以用来比较值和类型是否都相等。

**隐式类型转换是一把双刃剑，使用它虽然可以写更少的代码但有时候会出现难以被发现的bug**。

## 三种类型转换

我们需要知道的第一个规则是：**在 JS 中只有 3 种类型的转换**

* to string
* to boolean
* to number

第二，**类型转换的逻辑在原始类型和对象类型上是不同的，但是他们都只会转换成上面 3 种类型之一**。

我们首先分析一下原始类型转换。

## String 类型转换

`String()` 方法可以用来显式将值转为字符串，隐式转换通常在有 `+` 运算符并且有一个操作数是 `string` 类型时被触发，如：

```js
String(123) // 显式类型转换

123 + '' // 隐式类型转换

```

所有原始类型转 String 类型

```js
String(123)  // '123'
String(-12.3)  // '-12.3'
String(null)  // 'null'
String(undefined)  // 'undefined'
String(true)  // 'true'
```

Symbol 类型转 String 类型是比较严格的，它只能被显式的转换

```js
String(Symbol('symbol'))  // 'Symbol(symbol)'

'' + Symbol('symbol')  // TypeError is thrown
```

## Boolean 类型转换

`Boolean()` 方法可以用来显式将值转换成 boolean 型。

隐式类型转换通常在逻辑判断或者有逻辑运算符时被触发`（|| && !）`。

```js
Boolean(2)    // 显示类型转换
if(2) {}      // 逻辑判断触发隐式类型转换
!!2           // 逻辑运算符触发隐式类型转换
2 || 'hello'  // 逻辑运算符触发隐式类型转换
```
<!-- more -->

注意: 逻辑运算符（比如 || 和 &&）是在内部做了 boolean 类型转换，但实际上返回的是`原始操作数的值`，即使他们都不是 boolean 类型。

```js
// 返回 number 类型 123，而不是 boolean 型 true
// 'hello' 和 '123' 仍然在内部会转换成 boolean 型来计算表达式
let x = 'hello' && 123  // x === 123
```

boolean 类型转换只会有 true 或者 false 两种结果。

```js
Boolean('')           // false
Boolean(0)            // false  
Boolean(-0)           // false
Boolean(NaN)          // false
Boolean(null)         // false
Boolean(undefined)     // false
Boolean(false)        // false
```

**任何不在上面列表中的值都会转换为 true**, 包括 `object, function, Array, Date` 等，`Symbol` 类型是真值，空对象和空数组也是真值。

```js
Boolean({})             // true
Boolean([])             // true
Boolean(Symbol())       // true
!!Symbol()              // true
Boolean(function() {})  // true
```

## Number 类型转换

和 `Boolean()`、`String()` 方法一样， `Number()` 方法可以用来显式将值转换成 number 类型。
number 的隐式类型转换是比较复杂的，因为它可以在下面多种情况下被触发。

* 比较操作（>, <, <=, >=）
* 按位操作（| & ^ ~）
* 算数操作（- + * / %）， 注意，当 + 操作存在任意的操作数是 string 类型时，不会触发 number 类型的隐式转换
* 一 元 + 操作
* 非严格相等操作（== 或者 != ），注意，== 操作两个操作数都是 string 类型时，不会发生 number 类型的隐式转换


```js
Number('123')    // 显示类型转换
+ '123'          //  隐式类型转换
123 != "456"    //  隐式类型转换
4 > "5"        //  隐式类型转换
5 / null      //  隐式类型转换
true | 0      //  隐式类型转换
```

接下来看一下原始类型显示转换 number 类型会发生什么

```js
Number(null)                   // 0
Number(undefined)              // NaN
Number(true)                   // 1
Number(false)                  // 0
Number(" 12 ")                 // 12
Number("-12.34")               // -12.34
Number("\n")                   // 0
Number(" 12s ")                // NaN
Number(123)                    // 123
```

当将一个字符串转换为一个数字时，引擎首先删除前尾空格、`\n、\t` 字符，如果被修剪的字符串不成为一个有效的数字，则返回 NaN。如果字符串为空，则返回 0。

Number() 方法对于 null 和 undefined 的处理是不同的， null 会转换为 0, undefined 会转换为 NaN

**不管是显式还是隐式转换都不能将 Symbol 类型转为 number 类型，当试图这样操作时，会抛出错误**。

```js
Number(Symbol('my symbol'))    // TypeError is thrown
+Symbol('123')                 // TypeError is thrown
```

这里**有 2 个特殊的规则**需要记住：

1. 当将 == 应用于 null 或 undefined 时，不会发生数值转换。null 只等于 null 或 undefined，不等于其他任何值。
```js
null == 0               // false, null is not converted to 0
null == null            // true
undefined == undefined  // true
null == undefined       // true
undefined == 0          // false
```
2. NaN 不等于任何值，包括它自己

```js
NaN === NaN  // false

if(value !== value) { console.log('the value is NaN') }
```

## object 类型转换

到这里我们已经深入了解了原始类型的转换，接下来我们来看一下 object 类型的转换。

当涉及到对象的操作比如：`[1] + [2,3]`，引擎首先会尝试将 object 类型转为原始类型，然后在将原始类型转为最终需要的类型，而且仍然只有 3 种类型的转换：`number, string, boolean`

最简单的情况是 boolean 类型的转换，任何非原始类型总是会转换成 true,无论对象或数组是否为空。

对象通过内部 `[[ToPrimitive]]` 方法转换为原始类型，该方法负责数字和字符串转换。

`[[ToPrimitive]]` 方法接受两个参数一个输入值和一个需要转换的类型`（Numer or String）`
number 和 string的转换都使用了对象的两个方法： `valueOf` 和 `toString`。这两个方法都在 `Object.prototype` 上被声明，因此可用于任何派生类，比如 `Date, Array`等。

> 通常上 [[ToPrimitive]] 算法如下：

* 如果输入的值已经是原始类型，直接返回这个值。
* 输入的值调用 toString() 方法，如果结果是原始类型，则返回。
* 输入的值调用 valueOf() 方法，如果结果是原始类型，则返回。
* 如果上面 3 个步骤之后，转换后的值仍然不是原始类型，则抛出 TypeError 错误。

number 类型的转换首先会调用 `valueOf()` 方法，如果不是原始值在调用 `toString()` 方法。 string 类型的转换则相反。

大多数 JS 内置对象类型的 valueOf() 返回这个对象本身，其结果经常被忽略，因为它不是一个原始类型。所以大多数情况下当 object 需要转换成 number 或 string 类型时最终都调用了 `toString()` 方法。

当运算符不同时，`[[ToPrimitive]]` 方法接受的转换类型参数也不相同。当存在 == 或者 + 运算符时一般会先触发 number 类型的转换再触发 string 类型转换。

在 JS 中你可以通过重写对象的 `toString 和 valueOf` 方法来修改对象到原始类型转换的逻辑。

## 答案解析


```js
true + false // 1  
// '+' 运算符会触发 number 类型转换对于 true 和 false

12 / "6" // 2       
//算数运算符会把字符串 ‘6’ 转为 number 类型

"number" + 15 + 3 // 'number153'
// '+' 运算符按从左到右的顺序的执行，所以优先执行 “number” + 15, 把 15 转为 string 类型，得到 “number15” 然后同理执行 “number15” + 3

15 + 3 + "number"  // "18number"
// 15 + 3 先执行，运算符两边都是 number 类型 ，不用转换，然后执行 18 + “number” 最终得到 “18number”

[1] > null // true
// ==> '1' > 0
// ==> 1 > 0
// ==> true
// 比较运算符 > 执行 number 类型隐式转换。

"foo" + + "bar" // "fooNaN"
// ==> "foo" + (+"bar")
// ==> "foo" + NaN
// ==> "fooNaN"
// 一元 + 运算符比二元 + 运算符具有更高的优先级。所以 + bar表达式先求值。一元加号执行字符串“bar” 的 number 类型转换。因为字符串不代表一个有效的数字，所以结果是NaN。在第二步中，计算表达式'foo' + NaN。

"true" == true // false
false == "false" // false
// == 运算符执行 number 类型转换，'true' 转换为 NaN， boolean 类型 true 转换为 1

null == "" // false
// null 不等于任何值除了 null 和 undefined

!!"false" == !!"true" // true
// !! 运算符将字符串 'true' 和 'false' 转为 boolean 类型 true, 因为不是空字符串，然后两边都是 boolean 型不在执行隐式转换操作。

["x"] == "x" // true
// == 运算符对数组类型执行 number 转换，先调用对象的 valueOf() 方法，结果是数组本身，不是原始类型值，所以执行对象的 toString() 方法，得到字符串 'x'

[] + null + 1 // 'null1'
// '+' 运算符执行 number 类型转换，先调用对象的 valueOf() 方法，结果是数组本身，不是原始类型值，所以执行对象的 toString() 方法，得到字符串 ''， 接下来执行表达式 '' + null + 1。

[1,2,3] == [1,2,3] // false
// 当运算符两边类型相同时，不会执行类型转换，两个数组的内存地址不一样，所以返回 false

0 || "0" && {}  // {}
// 逻辑运算符 || 和 && 将值转为 boolean 型，但是会返回原始值（不是 boolean）。

{} + [] + {} + [1] // '0[object Object]1'

// 所有的操作数都不是原始类型，所以会按照从左到右的顺序执行 number 类型的隐式转换， object 和 array 类型的 valueOf() 方法返回它们本身，所以直接忽略，执行 toString() 方法。 这里的技巧是，第一个 {} 不被视为 object，而是块声明语句，因此它被忽略。计算从 +[] 表达式开始，该表达式通过toString()方法转换为空字符串，然后转换为0。

! + [] + [] + ![] // 'truefalse'
// 一元运算符优先执行，+[] 转为 number 类型 0，![] 转为 boolean 型 false。

new Date(0) - 0 // 0
// '-' 运算符执行 number 类型隐式转换对于 Date 型的值，Date.valueOf() 返回到毫秒的时间戳。

new Date(0) + 0 //"Thu Jan 01 1970 08:00:00 GMT+0800 (中国标准时间)0"
// +' 运算符触发默认转换，因此使用 toString() 方法，而不是 valueOf()。

```
