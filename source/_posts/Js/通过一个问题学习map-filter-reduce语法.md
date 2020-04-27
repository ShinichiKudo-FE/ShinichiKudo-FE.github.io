---
title: '通过一个问题学习map,filter,reduce语法'
date: 2019-07-24 09:44:52
tags: "Js"
categories: "Js"
---

## 比如一个从1-10的简单的数组，需要从中取奇数，你会怎么做？

这个问题不难，有的少侠可能会使用for循环来过滤出偶数，某些少侠可能会使用while循环，又或是使用数组自带的filter方法。

这里，我们先不考虑使用for和while这些迭代方法，我们选择使用数组自带的filter方法，如果使用filter方法的话，

```js
const nums = [1,2,3,4,5,6,7,8,9,10];
// 过滤出所有奇数
nums.filter(num => num % 2 !== 0);

console.log(nums);
```

还是挺简单的对吧？

相信大部分少侠都能很轻松完成。

在filter函数内部，我们使用 `num % 2 !== 0` 来判断 是否是一个奇数，如果是一个奇数，就过滤出来。

在这里，有的少侠可能就会想到，如果把这个判断过程单独分成一个函数，会方便一些，也方便以后的修改扩展。

比如单独写一个isOdd函数

```js
const isOdd = num => num % 2 !== 0;
```

然后在filter里面使用这个函数，是吧？

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
// 过滤出所有奇数
const odd = nums.filter(isOdd);

console.log(odd);
```

## 如果要把nums里面的数字翻倍，用map函数，又该怎么做呢？

聪明的少侠这次肯定一下就明白了：

```js
const nums = [1,2,3,4,5,6,7,8,9,10];
const double = num => num *2;
const doubleNumbers = nums.map(double);
console.log(doubleNumbers);

```

## 从nums里面过滤出所有奇数，并翻倍这些奇数。

这个问题也不难是吧？

让我猜一下，一些少侠应该会写成下面这样：

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const double = num => num *2;

const doubleNums = nums.filter(isOdd).map(double);

console.log(doubleNums);
```

## 如果我们还想继续算出所有奇数翻倍后的和的话

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const double = num => num *2;
const add = (sum,num) => sum += num;

const doubleNums = nums.filter(isOdd).map(double).reduce(add,0);

console.log(doubleNums);
```

嗯，一个串一个看起来很酷是吧？ 最后的结果也是正确的。

这样串起来的后果就是，我们会遍历3次数组，filter的时候遍历一次，map的时候遍历一次，reduce的时候再遍历一次。

现在数组只有10个数字，影响还不大，但是如果是很大的数组，比如100万个数字，遍历3遍的话，那么就会多访问**150万次**，

想象一下，如果让你自己从中间取出所有奇数，翻倍，并求和，你会觉得怎么做简单一些呢？

> 是不是觉得下面这样才比较正常？

* 首先，从第一个数字开始，如果它是奇数，就拿出来，翻倍，然后找个地方放起来
* 接着，继续看第二个数字，如果是偶数，就跳过，如果是奇数，就拿出来，翻倍，然后和开始的翻倍后的奇数加起来，找个地方放起来。
* 接着看第三个数字，以此类推，只要是奇数，就翻倍，然后和之前的结果加起来。
* 直到我们找寻到最后一个数字，就可以算出最终结果了，

如果使用迭代的话，就是下面这样:
<!-- more -->

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const double = num => num *2;
const add = (sum,num) => sum += num;

// 保存最终结果

let result = 0;

for (let i =0; i< nums.length; i++){
    if(nums[i]%2 != 0){
        result = result + nums[i]*2;
    }
}

console.log(result); // 50

```

没错，通常这样才是正常的操作。

但是，

这样的代码不太方便，

少侠你肯定也不想每次都写这么一大堆for循环语句吧？

所以，我们可以先试着继续优化一下。

我们首先试着用一个叫做magic(魔法)的函数把这些步骤包裹起来:

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const magic = array => {
    const length = array.length,
    let i = 0;
    let sum = 0;
    for(i, i < length; i++){
        if(array[i] % 2 !== 0){
            sum = sum + array[i]*2;
        }
    }
    return sum;
}
console.log(magic(nums)); // 50
```

现在看起来好多了，不过，magic内部的操作看起来不是很清晰，

其他少侠看见了可能很难一眼就明白magic函数在做什么,

我们可以试着把里面的一些步骤分成更小的函数:

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const double = num => num *2;
const add = (sum,num) => sum += num;
const magic = array => {
    const length = array.length,
    let i = 0;
    let sum = 0;
    for(i, i < length; i++){
        if(isOdd(array[i])){
            sum = add(sum,double(array[i]));
        }
    }
    return sum;
}
console.log(magic(nums)); // 50

```

现在比较简洁也比较清晰了，不过，假如我们现在要计算是偶数而不是奇数怎么办呢？

或者如果更复杂些，如果是偶数，就反过来，除以二，然后求和，又该怎么办呢？

难道我们又要复制一次magic函数，然后改动for循环里面的逻辑吗？

也许，我们可以把for循环内部的逻辑单独提取成一个外部函数！

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const isEvent = num => !isOdd(num)
const double = num => num *2;
const part = num => num /2;
const add = (sum,num) => sum += num;
const magic = (array,fn,sum) => {
    const length = array.length,
    let i = 0;
    for(i, i < length; i++){
        sum = magicFriends(sum,array[i])
    }
    return sum;
}

function magicFriends(sum,num){
    if(isOdd(num)){
        sum = add(sum,double(num));
    }
    return sum ;
}
console.log(magic(nums,magicFriends,0)); // 50

```

## 接下来就是见证奇迹的时刻！

首先，我们给代码里面部分函数和参数改一下名字:

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const isEvent = num => !isOdd(num)
const double = num => num *2;
const part = num => num /2;
const add = (sum,num) => sum += num;
const reduce = (array,reducer,result) => {
    const length = array.length,
    let i = 0;
    for(i, i < length; i++){
        result = reducer(sum,array[i])
    }
    return result;
}

consot magicFriends = (sum,num) =>{
    if(isOdd(num)){
        sum = add(sum,double(num));
    }
    return sum ;
}
console.log(reduce(nums,magicFriends,0)); // 50

```

是不是有点眼熟了？ 接着，我们换成数组自带的reduce函数:

```js
const nums = [1,2,3,4,5,6,7,8,9,10];

const isOdd = num => num % 2 !== 0;
const isEvent = num => !isOdd(num)
const double = num => num *2;
const part = num => num /2;
const add = (sum,num) => sum += num;
consot magicFriends = (sum,num) =>{
    if(isOdd(num)){
        sum = add(sum,double(num));
    }
    return sum ;
}
console.log(nums.reduce(magicFriends,0)); // 50
```

## 最后回到我们上面的问题:

```js
const nums = [1,2,3,4,5,6,7,8,9,10];
const isOdd = num => num % 2 !== 0;
const double = num => num *2;
const add = (sum,num) => sum += num;

const result = nums.reduces((sum,num)=> isOdd(num) ? add(sum,dobule(num) : sum,0));
console.log(result); // 50
```

这次我们用reduce只遍历了一次数组就完成了之前的操作！

现在知道为什么之前要把reduce叫做magic(魔法)了吧！