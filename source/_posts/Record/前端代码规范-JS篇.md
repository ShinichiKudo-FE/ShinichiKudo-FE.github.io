---
title: 前端代码规范-JS篇
date: 2018-04-19 00:21:34
tags: "Record"
categories: "Standard"
---
[更多前端代码规范请移步](https://github.com/ShinichiKudo-FE/spec)

-------

如果是个人或者小作坊开发，其实这些所谓的编码规范也没啥意思，因为大家写好的代码直接就给扔到网上去了，很少有打包、压缩、校检等过程，别人来修改你代码的情况也比较少。但是，对于一定规模的团队来说，这些东西还是挺有必要的！一个是保持代码的整洁美观，同时良好的代码编写格式和注释方式可以让后来者很快地了解你代码的大概思路，提高开发效率。

# 不规范写法举例
1. 句尾没有分号
```javascript
var isHotel = json.type == "hotel" ? true : false
```
2. 变量命名各种各样
```javascript
var is_hotel;
var isHotel;
var ishotel;
```
3. if 缩写
```javascript
if (isHotel)
    console.log(true)
else
    console.log(false)
```
4. 使用 eval
```javascript
var json = eval(jsonText);
```
5. 变量未定义到处都是
```javascript
function() {
    var isHotel = 'true';
    .......

    var html = isHotel ? '<p>hotel</p>' : "";
}
```
6. 超长函数
```javascript
function() {
    var isHotel = 'true';
    //....... 此处省略500行
    return false;
}
```
<!-- more -->
# 前端规范之JavaScript
## 变量、常量、类的命名按（必须）以下规则执行：

　　1） 变量：必须采用骆驼峰的命名且首字母小写
```javascript
 // 正确的命名
  var isHotel,
      isHotelBeijing,
      isHotelBeijingHandian;

  // 不推荐的命名
  var is_Hotel,
      ishotelbeijing,
      IsHotelBeiJing;
```
　　2） 常量：必须采用全大写的命名，且单词以_分割，常量通常用于ajax请求url，和一些不会改变的数据
```javascript
// 正确的命名
  var HOTEL_GET_URL = 'http://map.baidu.com/detail',
      PLACE_TYPE = 'hotel';
```
　　3） 类：必须采用骆驼峰的命名且首字母大写，如：
```javascript
 // 正确的写法
  var FooAndToo = function(name) {
      this.name = name;
  }
```

## if中的空格，先上例子
```javascript
 //正确的写法
  if (isOk) {
      console.log("ok");
  }

  //不推荐的写法
  if(isOk){
      console.log("ok");
  }
  ```
* ()中的判断条件前后都(必须)加空格
* ()里的判断前后(禁止)加空格，如：正确的写法: if (isOk)；不推荐的写法: if ( isOk )
　　2）switch中的空格, 先上例子
```javascript
//正确的写法
  switch(name) {
      case "hotel":
          console.log(name);
          break;

      case "moive":
          console.log(name);
          break;

      default:
          // code
  }  
```

## for中的空格，先上例子
```javascript
 // 正确的写法
  var names = ["hotel", "movie"],
      i, len;

  for (i=0, len=names.length; i < len; i++) {
      // code
  }  
``` 

* for后（必须）加空格
* 每个;后（必须）加空格
* ()中禁止用var声明变量; 且变量的赋值 = 前后（禁止）加空格

## function 中的空格, 先上例子
```javascript
 // 正确的写法
  function call(name) {

  }

  var cell = function () {

  };
```
* 参数的反括号后（必须）加空格
* function 后（必须）加空格

## var 中空格及定义，先上例子
```javascript
// 一个推荐的var写法组
  function(res) {
      var code = 1 + 1,
          json = JSON.parse(res),
          type, html;

      // code
  }
```

* 声明变量 = 前后（必须）添加空格
* 每个变量的赋值声明以,结束后（必须）换行进行下一个变量赋值声明
*（推荐）将所有不需要进行赋值的变量声明放置最后一行，且变量之间不需要换行
*（推荐）当一组变量声明完成后，空一行后编写其余代码

## 在同一个函数内部，局部变量的声明必须置于顶端

因为即使放到中间，js解析器也会提升至顶部（hosting）
```javascript
 // 正确的书写
 var clear = function(el) {
     var id = el.id,
         name = el.getAttribute("data-name");

     .........
     return true;
 }

 // 不推荐的书写
 var clear = function(el) {
     var id = el.id;

     ......

     var name = el.getAttribute("data-name");

     .........
     return true;
 }
 ```

##  块内函数必须用局部变量声明
```javascript
// 错误的写法
 var call = function(name) {
     if (name == "hotel") {
         function foo() {
             console.log("hotel foo");
         }
     }

     foo && foo();
 }

 // 推荐的写法
 var call = function(name) {
     var foo;

     if (name == "hotel") {
         foo = function() {
             console.log("hotel foo");
         }
     }

     foo && foo();
 }
``` 
引起的bug：第一种写法foo的声明被提前了; 调用call时：第一种写法会调用foo函数，第二种写法不会调用foo函数

注：不同浏览器解析不同，具体请移步汤姆大叔[深入解析Javascript函数篇](http://www.cnblogs.com/TomXu/archive/2012/01/30/2326372.html)

## 禁止）使用eval，采取$.parseJSON

> 三个原因：
有注入风险，尤其是ajax返回数据
不方便debug
效率低，eval是一个执行效率很低的函数


建议：使用new Function来代替eval的使用，最好就别用。

## 除了三目运算，if,else等（禁止）简写
```javascript
 // 正确的书写
 if (true) {
     alert(name);
 }
 console.log(name);
 // 不推荐的书写
 if (true)
     alert(name);
 console.log(name);
 // 不推荐的书写
 if (true)
 alert(name);
 console.log(name)
 ```

## （推荐）在需要以{}闭合的代码段前增加换行，如：for if
```java
 // 没有换行，小的代码段无法区分
 if (wl && wl.length) {
     for (i = 0, l = wl.length; i < l; ++i) {
         p = wl[i];
         type = Y.Lang.type(r[p]);
         if (s.hasOwnProperty(p)) {
             if (merge && type == 'object') {
                 Y.mix(r[p], s[p]);
             } else if (ov || !(p in r)) {
                 r[p] = s[p];
             }
         }
     }
 }
 // 有了换行，逻辑清楚多了
 if (wl && wl.length) {

     for (i = 0, l = wl.length; i < l; ++i) {
         p = wl[i];
         type = Y.Lang.type(r[p]);

         if (s.hasOwnProperty(p)) {
             // 处理merge逻辑
             if (merge && type == 'object') {
                 Y.mix(r[p], s[p]);
             } else if (ov || !(p in r)) {
                 r[p] = s[p];
             }
         }
     }
 }
 ```
换行可以是空行，也可以是注释

## (推荐）使用Function进行类的定义，(不推荐)继承，如需继承采用成熟的类库实现继承
```javascript
// 类的实现
 function Person(name) {
     this.name = name;
 }

 Person.prototype.sayName = function() {
     alert(this.name);
 };

 var me = new Person("Nicholas");

 // 将this放到局部变量self
 function Persion(name, sex) {
     var self = this;

     self.name = name;
     self.sex = sex;
 }
 ```
 平时咱们写代码，基本都是小程序，真心用不上什么继承，而且继承并不是JS的擅长的语言特性，尽量少用。如果非要使用的话，注意一点：
```java
 function A(){
    //...
}
function B(){
    //...
}
B.prototype = new A();
B.prototype.constructor = B; //原则上，记得把这句话加上
```
 继承从原则上来讲，别改变他的构造函数，否则这个继承就显得很别扭了~ 

## (推荐)使用局部变量缓存反复查找的对象(包括但不限于全局变量、dom查询结果、作用域链较深的对象)
```java
 // 缓存对象
 var getComment = function() {
     var dom = $("#common-container"),               // 缓存dom
         appendTo = $.appendTo,                      // 缓存全局变量
         data = this.json.data;                      // 缓存作用域链较深的对象

 }
``` 

## 当需要缓存this时必须使用self变量进行缓存
```java
// 缓存this
 function Row(name) {
     var self = this;

     self.name = name;
     $(".row").click(function() {
         self.getName();
     });
 }
 ```
 self是一个保留字，不过用它也没关系。在这里，看个人爱好吧，可以用_this, that, me等这些词，都行，但是团队开发的时候统一下比较好。

##（不推荐）超长函数, 当函数超过100行，就要想想是否能将函数拆为两个或多个函数

[原文地址](http://www.cnblogs.com/hustskyking/p/javascript-spec.html)