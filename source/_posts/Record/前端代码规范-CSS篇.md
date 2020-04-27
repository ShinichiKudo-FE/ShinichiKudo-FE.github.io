---
title: 前端代码规范-CSS篇
date: 2018-04-18 16:41:22
tags: "Record"
categories: "Standard"
---

## tab键用（必须）四个空格代替

　　因为在不同系统的编辑工具对tab解析不一样，windows下的tab键是占四个空格的位置，而在linux下会变成占八个空格的位置（除非你自己设定了tab键所占的位置长度）。

## 每个样式属性后（必须）加 ";"

    方便压缩工具"断句"。

## Class命名中（禁止）出现大写字母，（必须）采用” - “对class中的字母分隔，如：
```css
 /* 正确的写法 */
 .hotel-title {
     font-weight: bold;
 }

 /* 不推荐的写法 */
 .hotelTitle {
     font-weight: bold;
 }
```    

* 用"-"隔开比使用驼峰是更加清晰。
* 产品线-产品-模块-子模块，命名的时候也可以使用这种方式（@Artwl）

## 空格的使用，以下规则（必须）执行：
```css
.hotel-content {
     font-weight: bold;
 }
```

* 选择器与 { 之前（必须）要有空格
* 属性名的 : 后（必须）要有空格
* 属性名的 : 前（禁止）加空格

## 多选择器规则之间（必须）换行

当样式针对多个选择器时每个选择器占一行

```css
 /* 推荐的写法 */
 a.btn,
 input.btn,
 input[type="button"] {
     ......
 }
 ```
<!-- more -->
## （禁止）将样式写为单行, 如
```css
.hotel-content {margin: 10px; background-color: #efefef;}
```
单行显示不好注释，不好备注，这应该是压缩工具的活儿~

## （禁止）向 0 后添加单位, 如：
```css
.obj {
    left: 0px;
}
```

## （禁止）使用css原生import

使用css原生import有很多弊端，比如会增加请求数等....

## （推荐）属性的书写顺序, 举个例子:
```css
.hotel-content {
     /* 定位 */
     display: block;
     position: absolute;
     left: 0;
     top: 0;
     /* 盒模型 */
     width: 50px;
     height: 50px;
     margin: 10px;
     border: 1px solid black;
     / *其他* /
     color: #efefef;
 }
```

* 定位相关, 常见的有：display position left top float 等
* 盒模型相关, 常见的有：width height margin padding border 等
* 其他属性

> 按照这样的顺序书写可见提升浏览器渲染dom的性能
　简单举个例子，网页中的图片，如果没有设置width和height，在图片载入之前，他所占的空间为0，但是当他加载完毕之后，那块为0的空间突然被撑开了，这样会导致，他下面的元素重新排列和渲染，造成重绘（repaint）和回流（reflow）。我们在写css的时候，把元素的定位放在前头，首先让浏览器知道该元素是在文本流内还是外，具体在页面的哪个部位，接着让浏览器知道他们的宽度和高度，border等这些占用空间的属性，其他的属性都是在这个固定的区域内渲染的，差不多就是这个意思吧~(@frec)

 推荐文章： https://css-tricks.com/poll-results-how-do-you-order-your-css-properties/
          http://www.mozilla.org/css/base/content.css

##  小图片（必须）sprite 合并

推荐文章：[NodeJs智能合并CSS精灵图工具iSpriter](http://www.alloyteam.com/2012/09/update-ispriter-smart-merging-css-sprite/)


## (推荐）当编写针对特定html结构的样式时，使用元素名 + 类名

/* 所有的nav都是针对ul编写的 */
```css
 ul.nav {
     ......
 }
```
".a div"和".a div.b"，为什么后者好？如果需求有所变化，在".a"下有多加了一个div，试问，开始的样式是不是会影响后来的div啊~

## (推荐）IE Hack List
```css
 /* 针对ie的hack */
 selector {
     property: value;     /* 所有浏览器 */ 
     property: value\9;   /* 所有IE浏览器 */ 
     property: value\0;   /* IE8 */
     +property: value;    /* IE7 */
     _property: value;    /* IE6 */
     *property: value;    /* IE6-7 */
 }
```
当使用hack的时候想想能不能用更好的样式代替

## （不推荐）ie使用filter,（ 禁止）使用expression

这里主要是效率问题，应该当格外注意，咱们要少用烧CPU的东西~ 

## (禁止）使用行内（inline）样式
```html
<p style="font-size: 12px; color: #FFFFFF">靖鸣君</p>
```
像这样的行内样式，最好用一个class代替。又如要隐藏某个元素，可以给他加一个class

.hide {
    display: none;
}
尽量做到样式和结构分离~

## （推荐）reset.css样式

推荐网站：http://www.cssreset.com/

## （禁止）使用"*"来选择元素
```css
/*别这样写*/
* {
    margin: 0;
    padding: 0;
}
```
这样写是没有必要的，一些元素在浏览器中默认有margin或padding值，但是只是部分元素，没有必要将所有元素的margin、padding值都置为0。

链接的样式，（务必）按照这个顺序来书写
```css
a:link -> a:visited -> a:hover -> a:active
```

[原文地址](http://www.cnblogs.com/hustskyking/p/css-spec.html)