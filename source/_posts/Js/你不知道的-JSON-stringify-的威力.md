---
title: 你不知道的 JSON.stringify() 的威力
date: 2019-12-12 16:17:03
tags: "Js"
categories: "Js"
---

[原文地址](https://juejin.im/post/5decf09de51d45584d238319#heading-4)

## 通过需求学习JSON.stringify()

首先我们在开发的过程当中遇到这样一个处理数据的需求

```js
const todayILearn = {
  _id: 1,
  content: '今天学习 JSON.stringify()，我很开心！',
  created_at: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
  updated_at: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
}
```

我们需要将上面这个对象处理成下面这个对象

```js
const todayILearn = {
  id: 1,
  content: '今天学习 JSON.stringify()，我很开心！',
  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
  updatedAt: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)'
}
```

也就是在不改变属性的值的前提下，将对象属性修改一下。 把_id  改成 id，把 updated_at 改成 updatedAt，把 created_at 改成 createdAt。我们现在通过这个小小的需求来见识一下 JSON.stringify() 的强大吧。


首先要解决这个问题我们有很多种解决方式，我们先提供两种不优雅的解决方案：

**方案一：一次遍历+多声明一个变量**

```js
// 多一个变量存储
const todayILearnTemp = {};
for (const [key, value] of Object.entries(todayILearn)) {
  if (key === "_id") todayILearnTemp["id"] = value;
  else if (key === "created_at") todayILearnTemp["createdAt"] = value;
  else if (key === "updatedAt") todayILearnTemp["updatedAt"] = value;
  else todayILearnTemp[key] = value;
}
console.log(todayILearnTemp);
// 结果：
// { id: 1,
//  content: '今天学习 JSON.stringify()，我很开心！',
//  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
//  updated_at: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)' 
// }
```

方案一完全没有问题，可以实现。但是多声明了一个变量又加上一层循环并且还有很多的 if else 语句，怎么都显得不太优雅。

**方案二：暴力 delete 属性和增加属性**

```js
// 极致的暴力美学
todayILearn.id = todayILearn._id;
todayILearn.createdAt = todayILearn.created_at;
todayILearn.updatedAt = todayILearn.updated_at;
delete todayILearn._id;
delete todayILearn.created_at;
delete todayILearn.updated_at;
console.log(todayILearn);
// 	太暴力😢
//{ 
//  content: '今天学习 JSON.stringify()，我很开心！',
//  id: 1,
//  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
//  updatedAt: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)' 
//}

```
直接 delete 暴力解决太粗鲁了，而且有一个缺点，属性的顺序变了。

**方案三：序列化+ replace 美学典范**

```js
const mapObj = {
  _id: "id",
  created_at: "createdAt",
  updated_at: "updatedAt"
};
JSON.parse(
  JSON.stringify(todayILearn).replace(
    /_id|created_at|updated_at/gi,
    matched => mapObj[matched])
    )

// { 
// id: 1,
//  content: '今天学习 JSON.stringify()，我很开心！',
//  createdAt: 'Mon Nov 25 2019 14:03:55 GMT+0800 (中国标准时间)',
//  updatedAt: 'Mon Nov 25 2019 16:03:55 GMT+0800 (中国标准时间)' 
// }
```

![Json.Stringify](https://user-gold-cdn.xitu.io/2019/12/8/16ee5ace18ca5dbb?imageslim)

<!-- more -->

## JSON.stringify() 九大特性

### JSON.stringify()第一大特性

**对于 undefined、任意的函数以及 symbol 三个特殊的值分别作为对象属性的值、数组元素、单独的值时 JSON.stringify()将返回不同的结果**。

首先，我们来复习一下知识点，看一道非常简单的面试题目：请问下面代码会输出什么？

```js
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
JSON.stringify(data); // 输出：？

// "{"a":"aaa"}"

```

很简单这道题目面试官主要考察的知识点是：

* undefined、任意的函数以及 symbol 作为对象属性值时 JSON.stringify() 将跳过（忽略）对它们进行序列化

面试官追问：假设 undefined、任意的函数以及 symbol 值作为数组元素会是怎样呢？

```js
JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd')])  // 输出：？

// "["aaa",null,null,null]"
```

知识点是：

* undefined、任意的函数以及 symbol 作为数组元素值时，JSON.stringify() 会将它们序列化为 null

我们稍微再动下脑筋，如果单独序列化这些值会是什么样的结果呢？

```js
JSON.stringify(function a (){console.log('a')})
// undefined
JSON.stringify(undefined)
// undefined
JSON.stringify(Symbol('dd'))
// undefined
```

单独转换的结果就是：

undefined、任意的函数以及 symbol 被 JSON.stringify() 作为单独的值进行序列化时都会返回 undefined

> JSON.stringify() 第一大特性总结
undefined、任意的函数以及 symbol 作为对象属性值时 JSON.stringify() 对跳过（忽略）它们进行序列化
undefined、任意的函数以及 symbol 作为数组元素值时，JSON.stringify() 将会将它们序列化为 null
undefined、任意的函数以及 symbol 被 JSON.stringify() 作为单独的值进行序列化时，都会返回 undefined

### JSON.stringify() 第二大特性

**非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。**

```js
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  },
  d: "ddd"
};
JSON.stringify(data); // 输出：？
// "{"a":"aaa","d":"ddd"}"

JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd'),"eee"])  // 输出：？

// "["aaa",null,null,null,"eee"]"
```

正如我们在第一特性所说，JSON.stringify() 序列化时**会忽略一些特殊的值**，所以不能保证序列化后的字符串还是以特定的顺序出现（数组除外）。

### JSON.stringify() 第三大特性

* 转换值如果**有 toJSON() 函数，该函数返回什么值，序列化结果就是什么值，并且忽略其他属性的值**。

```js
JSON.stringify({
    say: "hello JSON.stringify",
    toJSON: function() {
      return "today i learn";
    }
  })
// "today i learn"
```

### JSON.stringify()第四大特性

JSON.stringify() 将会**正常序列化 Date** 的值。

```js
JSON.stringify({ now: new Date() });
// "{"now":"2019-12-08T07:42:11.973Z"}"
```

实际上 Date 对象自己部署了 toJSON() 方法（同Date.toISOString()），因此 Date 对象会被当做字符串处理。

### JSON.stringify() 第五大特性

**NaN 和 Infinity 格式的数值及 null 都会被当做 null。**

```js
JSON.stringify(NaN)
// "null"
JSON.stringify(null)
// "null"
JSON.stringify(Infinity)
// "null"
```

### JSON.stringify() 第六大特性

**布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。**

```js
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]);
// "[1,"false",false]"
```

### JSON.stringify() 第七大特性

关于对象属性的是否可枚举：

**其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。**

```js
// 不可枚举的属性默认会被忽略：
JSON.stringify( 
    Object.create(
        null, 
        { 
            x: { value: 'json', enumerable: false }, 
            y: { value: 'stringify', enumerable: true } 
        }
    )
);
// "{"y":"stringify"}"
```

### JSON.stringify() 第八大特性

我们都知道实现深拷贝最简单粗暴的方式就是序列化：`JSON.parse(JSON.stringify())`，这个方式实现深拷贝会因为序列化的诸多特性从而导致诸多的坑点：比如现在我们要说的循环引用问题。

```js
// 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。 
const obj = {
  name: "loopObj"
};
const loopObj = {
  obj
};
// 对象之间形成循环引用，形成闭环
obj.loopObj = loopObj;

// 封装一个深拷贝的函数
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}
// 执行深拷贝，抛出错误
deepClone(obj)
/**
 VM44:9 Uncaught TypeError: Converting circular structure to JSON
    --> starting at object with constructor 'Object'
    |     property 'loopObj' -> object with constructor 'Object'
    --- property 'obj' closes the circle
    at JSON.stringify (<anonymous>)
    at deepClone (<anonymous>:9:26)
    at <anonymous>:11:13
 */
```

* 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。

这也就是为什么用序列化去实现深拷贝时，遇到循环引用的对象会抛出错误的原因

### JSON.stringify() 第九大特性

**所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们。**

```js
JSON.stringify({ [Symbol.for("json")]: "stringify" }, function(k, v) {
    if (typeof k === "symbol") {
      return v;
    }
  })

// undefined
```

`replacer` 是 `JSON.stringify()` 的第二个参数，我们比较少用到，所以很多时候我们会忘记 `JSON.stringify()` 第二个、第三个参数，场景不多，但是用的好的话会非常方便，关于 `JSON.stringify()` 第九大特性的例子中对 replacer 参数不明白的同学先别急，其实很简单，我们马上就会在下面的学习中弄懂。

## 第二个参数和第三个参数

JSON.stringify() 第二个参数和第三个参数
### 强大的第二个参数：

作为函数时，它有两个参数，键（key）和值（value），函数类似就是数组方法 map、filter 等方法的回调函数，对每一个属性值都会执行一次该函数（期间我们还简单实现过一个 map 函数）。
如果 replacer 是一个数组，数组的值代表将被序列化成 JSON 字符串的属性名。

### 华丽的第三个参数：

如果是一个数字, 则在字符串化时每一级别会比上一级别缩进多这个数字值的空格（最多10个空格）；

如果是一个字符串，则每一级别会比上一级别多缩进该字符串（或该字符串的前10个字符）。
