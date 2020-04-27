---
title: 移动端常见Bug汇总
date: 2018-06-08 10:52:04
tags: "Mobile"
categories: "Bug"
---

# 前言

本文是摘录整理了移动端常见的一些bug以及解决方案

## 点击样式闪烁

Q: 当你点击一个链接或者通过Javascript定义的可点击元素的时候，它就会出现一个半透明的灰色背景。

A:根本原因是-webkit-tap-highlight-color，这个属性是用于设定元素在移动设备（如Adnroid、iOS）上被触发点击事件时，响应的背景框的颜色。建议写在样式初始化中以避免所以问题：div,input(selector) {-webkit-tap-highlight-color: rgba(0,0,0,0);}另外出现蓝色边框：outline:none；

```js
-webkit-tap-highlight-color : rgba (255, 255, 255, 0) ;
// i.e . Nexus5/Chrome and Kindle Fire HD 7 ''
-webkit-tap-highlight-color : transparent ;  
```

## 屏蔽用户选择

Q: 禁止用户选择页面中的文字或者图片

A:代码如下

```css
-webkit-touch-callout: none;
-webkit-user-select: none;
-khtml-user-select: none;
-moz-user-select: none;
-ms-user-select: none;
user-select: none;
```

## 移动端如何清除输入框内阴影

Q: 在iOS上，输入框默认有内部阴影，但无法使用 box-shadow 来清除，如果不需要阴影，可以这样关闭：

A:代码如下

```css
-webkit-appearance: none;
```

## 禁止文本缩放

Q: 禁止文本缩放

A:代码如下

```css
-webkit-text-size-adjust: 100%;
```

<!-- more -->

## 如何禁止保存或拷贝图像

Q: 如何禁止保存或拷贝图像

A:代码如下

```css
img{
-webkit-touch-callout: none;}
```

## 解决字体在移动端比例缩小后出现锯齿的问题

Q: 解决字体在移动端比例缩小后出现锯齿的问题

A:代码如下

```css
-webkit-font-smoothing: antialiased;
```

## 设置input里面placeholder字体的大小

Q: 设置input里面placeholder字体的大小

A:代码如下

```css
::-webkit-input-placeholder{ font-size:10pt;}
```

## audio元素和video元素在ios和andriod中无法自动播放

Q: audio元素和video元素在ios和andriod中无法自动播放

A:代码如下,触屏及播放

```js
$('html').on('touchstart',function(){
audio.play()
})
```

## 手机拍照和上传图片

Q: 针对file类型增加不同的accept字段

A:代码如下

```html
<input type="file">的accept 属性
<!-- 选择照片 -->
<input type=file accept="image/*">
<!-- 选择视频 -->
<input type=file accept="video/*">
```

## 输入框自动填充颜色

Q: 针对input标签已经输入过的，会针对曾经输入的内容填充黄色背景，这是webkit内核自动添加的，对应的属性是autocomplete,默认是on,另对应的样式是input:-webkit-autofill 且是不可更改的。

A:方案如下 1 设置标签的autocomplete="off",亲测无效可能 2 设置盒子的内阴影为你常态的颜色（下面以白色为例）

```css
 box-shadow:0 0  0 1000px  #fff inset ;
 -webkit-box-shadow: 0 0 0px 1000px #fff inset;
```

## 开启硬件加速

Q: 优化渲染性能

A:代码如下

```css
-webkit-transform: translate3d(0, 0, 0);
-moz-transform: translate3d(0, 0, 0);
-ms-transform: translate3d(0, 0, 0);
transform: translate3d(0, 0, 0);
```

## 用户设置字号放大或者缩小导致页面布局错误

```css
body  
    {  
        -webkit-text-size-adjust: 100% !important;  
        text-size-adjust: 100% !important;  
        -moz-text-size-adjust: 100% !important;  
    }
```

## 移动端去除type为number的箭头

```css
 input::-webkit-outer-spin-button,input::-webkit-inner-spin-button{
      -webkit-appearance: none !important;
      margin: 0;
  }
```

## 实现横屏竖屏的方案

### css 用 css3媒体查询，缺点是宽度和高度不好控制

```css
@media screen and (orientation: portrait) {
    .main {
        -webkit-transform:rotate(-90deg);
        -moz-transform: rotate(-90deg);
        -ms-transform: rotate(-90deg);
        transform: rotate(-90deg);
        width: 100vh;
        height: 100vh;
        /*去掉overflow 微信显示正常，但是浏览器有问题，竖屏时强制横屏缩小*/
        overflow: hidden;
    }
}

@media screen and (orientation: landscape) {
    .main {
        -webkit-transform:rotate(0);
        -moz-transform: rotate(0);
        -ms-transform: rotate(0);
        transform: rotate(0)
    }
}
```

### js 判断屏幕的方向或者resize事件

```js
var evt = "onorientationchange" in window ? "orientationchange" : "resize";
    window.addEventListener(evt, function() {
        var width = document.documentElement.clientWidth;
         var height =  document.documentElement.clientHeight;
          $print =  $('#print');
         if( width > height ){

            $print.width(width);
            $print.height(height);
            $print.css('top',  0 );
            $print.css('left',  0 );
            $print.css('transform' , 'none');
            $print.css('transform-origin' , '50% 50%');
         }
         else{
            $print.width(height);
            $print.height(width);
            $print.css('top',  (height-width)/2 );
            $print.css('left',  0-(height-width)/2 );
            $print.css('transform' , 'rotate(90deg)');
            $print.css('transform-origin' , '50% 50%');
         }

    }, false);
```

[原文地址](https://juejin.im/post/5af918636fb9a07ac5603ecb)