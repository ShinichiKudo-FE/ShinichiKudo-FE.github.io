---
title: jQuery 与 Zepto的区别
date: 2018-04-13 15:32:20
tags: "jQuery"
categories: "Js"
---

> jQuery 的意义是在于抹平 DOM、BOM、CSSOM 的多浏览器差异，并统一提供接口。它不能当作 Framework（框架）被使用，而是扮演 Utility（工具）的角色。

虽然 Zepto 和 jQuery 的中的很多 API 的用法是相似的，但仍有一些 API 在用法上差异很大。下面就实际使用中遇到的场景做一下列举。

## 创建DOM节点并赋与属性

使用 `$` 操作符可以轻松创建节点并赋予属性，具体代码是 `$(htmlString, attributes)` ，支持链式写法。喜欢刨根问底的同学可以看看下面的文档。

[jquert文档](http://api.jquery.com/jQuery/#jQuery-html-attributes)
[zepto文档](http://zeptojs.com/#)

### DOM操作的区别

**jquery操作ul元素时候，添加id不会生效**

```javascript
(function($){
    $(function){
         var $list = $('<ul><li>jQuery 插入</li></ul>', {
            id: 'insert-by-jquery'
        });
        $list.appendTo($('body'));
    }
})(window.jQuery)
// Result
// <ul><li>jQuery 插入</li></ul>
```
**zepto操作ul元素时候，添加id会生效**

```javascript
zepto(function($){
 
         var $list = $('<ul><li>jQuery 插入</li></ul>', {
            id: 'insert-by-jquery'
        });
        $list.appendTo($('body'));
})
// Result
// <ul id="insert-by-zepto"><li>Zepto 插入</li></ul>
```
<!-- more -->
### 事件触发的区别

**jquery时load事件的处理函数不会执行**
```javascript
(function($) {
    $(function() {    
        $script = $('<script />', {
            src: 'http://cdn.amazeui.org/amazeui/1.0.1/js/amazeui.js',
            id: 'ui-jquery'
        });

        $script.appendTo($('body'));

        $script.on('load', function() {
            console.log('jQ script loaded');//没有打印出
        });
    });
})(window.jQuery);
```
**使用 Zepto 时 load 事件的处理函数会执行**
```javascript
Zepto(function($) {  
    $script = $('<script />', {
        src: 'http://cdn.amazeui.org/amazeui/1.0.1/js/amazeui.js',
        id: 'ui-zepto'
    });

    $script.appendTo($('body'));

    $script.on('load', function() {
        console.log('zepto script loaded'); //zepto script loaded
    });
});
```

### 事件委托的区别
**在 Zepto 中，当 a 被点击后，依次弹出了内容为”a事件“和”b事件“的弹出框。说明虽然事件委托在.a上可是却也触发了.b上的委托。**

**但是在 jQuery 中只会触发 .a 上面的委托。**
```javascript
var $doc = $(document);

// Class 'a' bind event 'a'
$doc.on('click', '.a', function () {
    alert('a事件');
    // Class 'a' change to class 'b'
    $(this).removeClass('a').addClass('b');
});

// Class 'b' bind event 'b'
$doc.on('click', '.b', function () {
    alert('b事件');
});
```

**原理分析**
```javascript
// Zepto source code
$.fn.on = function(event, selector, data, callback, one) {
    var autoRemove, delegator, $this = this
    if (event && !isString(event)) {
        // Core code
        $.each(event, function (type, fn) {
            $this.on(type, selector, data, fn, one)
        }) return $this
    }
    //...
}
```
> 在 Zepto 中代码解析的时候，document上所有的click委托事件都依次放入到一个队列中，点击的时候先看当前元素是不是.a，符合则执行，然后查看是不是.b，符合则执行。

这样的话，就导致如果.a的事件在前面，会先执行.a事件，然后class更改成b，Zepto再查看当前元素是不是.b，以此类推。

> 在 jQuery 中代码解析的时候，document上委托了2个click事件，点击后通过选择符进行匹配，执行相应元素的委托事件。

这样就很好的避免了在 Zepto 中的发生的情况。

## API方面的区别

### `winth()` 和 `height`的区别

> * Zepto 由盒模型(box-sizing)决定；

> * jQuery 会忽略盒模型，始终返回内容区域的宽/高(不包含padding、border)。


解决方式就是使用 `.css('width')` 而不是`.width()` 。下面我们举个叫做《边框三角形宽高的获取》的栗子来说明这个问题。

首先用下面的 HTML 和 CSS 画了一个小三角形吧。
```javascript
// html
<div class="caret"></div>

// css
.caret{
    width:0;
    height:0;
    border-width: 0 20px 20px;
    border-color: transparent transparent blue; 
    border-style: none dotted solid;
}

```
> * jQuery 使用 .width() 和 .css('width') 都返回 ，高度也一样；

> * Zepto 使用 .width() 返回 ，使用 .css('width') 返回 0px 。
所以，这种场景，jQuery 使用 `.outerWidth() /` .outerHeight() ；Zepto 使用 `.width() / .height()` 。

### offSet()的区别

```javascript
// offset() {function}

// @desc Zepto
// @return top|left|width|height

// @desc jQuery
// @return width|height
```

### 获取隐藏元素width和height的区别

Zepto 无法获取隐藏元素宽高，jQuery 可以。

### extend() 的区别

**jQuery 在原型上拓展方法使用的方式是：**

```javascript
// For example
// Extend a function named 'sayHello' to the protorype of jQuery
(function($) {
    $.fn.extend({
        sayHello: function() {
            $(this).html('Hello !');
        }
    });
})(window.jQuery);
```

**Zepto 中并没有为原型定义extend方法，所以如果要是要在 Zepto 的原型上拓展方法可以使用的方式是：**

```javascript
// For example
// Extend a function named 'sayHello' to the protorype of Zepto
Zepto(function($) {
    sayHello: function() {
        $(this).html('Hello !');
    }
});
```
随着 jQuery 2.x 的发布以及未来 3.0 对浏览器支持的划分，似乎找不到再使用 Zepto 的理由了。如果你真在乎文件大小，那你可以自行打包 jQuery 中需要的模块。这和 Zepto 是一样的，Zepto 官方提供的版本只打包了很基础的模块。

[原文地址](https://segmentfault.com/a/1190000003409961)
