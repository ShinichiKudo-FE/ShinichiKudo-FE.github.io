---
title: Ajax原理与实现
date: 2018-05-04 09:29:36
tags: "Ajax"
categories: "原理与实现"
---
# 概述

AJAX全称为“Asynchronous JavaScript and XML”（异步JavaScript和XML），是一种创建交互式网页应用的网页开发技术。

## 原理与实现

> 原理：Ajax的原理简单来说通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。

把服务器端看成一个数据接口(只负责吐数据)，它返回的是一个`纯文本流`，当然，这个文本流可以是`XML格式`，可以是`Html`，可以是`Javascript代码`，也可以只是一个字符串。这时候，`XMLHttpRequest`向服务器端请求这个页面，服务器端将文本的结果写入页面，这和普通的web开发流程是一样的，不同的是，<font color = "red">客户端在异步获取这个结果后，不是直接显示在页面，而是先由javascript来处理，然后再显示在页面</font>。

### 核心

Ajax其核心有JavaScript、XMLHTTPRequest、DOM对象组成，XMLHttpRequest是ajax的核心机制，是JavaScript的一个内置对象。它是在IE5中首先引入的，是一种支持异步请求的技术。简单的说，也就是javascript可以及时向服务器提出请求和处理响应，而不阻塞用户。达到无刷新的效果。

### XMLHTTPRequest对象

Ajax的一个最大的特点是无需刷新页面便可向服务器传输或读写数据(又称无刷新更新页面),这一特点主要得益于XMLHTTP组件XMLHTTPRequest对象。
<!-- more -->
**XMLHttpRequest 对象方法描述**
Ajax引擎，实际上是一个比较复杂的JavaScript应用程序，用来处理用户请求，读写服务器和更改DOM内容。JavaScript的Ajax引擎读取信息，并且互动地重写DOM，这使网页能无缝化重构，也就是在页面已经下载完毕后改变页面内容，这是我们一直在通过JavaScript和DOM在广泛使用的方法，但要使网页真正动态起来，不仅要内部的互动，还需要从外部获取数据，在以前，我们是让用户来输入数据并通过DOM来改变网页内容的，但现在，XMLHTTPRequest，可以让我们在不重载页面的情况下读写服务器上的数据，使用户的输入达到最少。

Ajax使WEB中的界面与应用分离（也可以说是数据与呈现分离），而在以前两者是没有清晰的界限的，数据与呈现分离的分离，有利于分工合作、减少非技术人员对页面的修改造成的WEB应用程序错误、提高效率、也更加适用于现在的发布系统。也可以把以前的一些服务器负担的工作转嫁到客户端，利于客户端闲置的处理能力来处理。

## 优点与缺点

### 优点

* 无需刷新页面更新数据
* 界面与应用分离（呈现与数据分离）
* 基于规范广泛使用
* 异步与服务器通信
* 前后端负载平衡

### 缺点

* 破坏了浏览器的back和history功能
* 搜索引擎的支持较弱
* ajax的安全问题（跨站点脚本攻击，SQL注入攻击等等）
* Ajax不能很好地支持移动端
* 破坏程序的异常处理机制
* 客户端过肥，容易造成客户端的开发成本

## AJAX注意点及适用和不适用场景

### 注意点

Ajax开发时，网络延迟——即用户发出请求到服务器发出响应之间的间隔——需要慎重考虑。不给予用户明确的回应，没有恰当的预读数据，或者对XMLHttpRequest的不恰当处理，都会使用户感到延迟，这是用户不希望看到的，也是他们无法理解的。通常的解决方案是，使用一个可视化的组件来告诉用户系统正在进行后台操作并且正在读取数据和内容。

### Ajax适用场景

* 表单驱动的交互
* 深层次的树的导航
* 快速的用户与用户间的交流响应
* 类似投票、yes/no等无关痛痒的场景
* 对数据进行过滤和操纵相关数据的场景
* 普通的文本输入提示和自动完成的场景

### Ajax不适用场景

* 部分简单的表单
* 搜索
* 基本的导航
* 替换大量的文本
* 对呈现的操纵

## 使用ES6去实现一个ajax

```js
function Ajax(method,url){
    const p =  new Promise((resolve,reject) =>{
        const xhr = new XMLHttpRequest();
        xhr.open(method,url,true)
        xhr.onReadyStateChange(data){
            if(xhr.readyStatus == 4){
                if(xhr.status == 200){
                    resolve(json.parse(xhr.responseText))
                }else{
                    reject(new throw ('not found'))
                }
            }
        }
        xhr.send(null)
    })
}

```