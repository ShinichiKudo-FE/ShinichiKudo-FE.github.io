---
title: 封装常用的跨浏览器的事件对象
date: 2018-05-01 21:28:34
tags: "Js"
categories: "Js"
---
# 前言

最近把《javascript高级程序设计》看完了，发现里面有很多跨浏览器的代码，总结一下，方便记录。

## 跨浏览器的事件对象EventUtil(事件处理程序)

```js
var EventUtil = {
    addHandler: function(element,type,handler) { //事件监听
        if(element.addEventListener) {
            element.addEventListener(type,handler,false);
        }else if(element.attachEvent) {
            element.attachEvent("on"+type,handler);
        }else {
            element["on" +type] = handler;
        }
    },
    removeHandler: function(element,type,handler){ //移除事件监听
        if(element.removeEventListener) {
            element.removeEventListener(type,handler,false);
        }else if(element.detachEvent) {
            element.detachEvent("on"+type,handler);
        }else {
            element["on" +type] = null;
        }
    },
    getEvent: function(event) {//获取event对象，返回event对象的引用
        return event ? event : window.event;
    },
    getTarget: function(event) {//返回事件目标。
        return event.target || event.srcElement;
    },
    preventDefault: function(event){//取消或者阻止事件默认行为
        if(event.preventDefault) {
            event.preventDefault();
        }else {
            event.returnValue = false;
        }
    },
    stopPropagation: function(event) {//阻止事件流，阻止事件冒泡
        if(event.stopPropagation) {
            event.stopPropagation();
        }else {
            event.cancelBubble = true;
        }
    },
    getRelatedTarget: function(event){//返回相关元素信息（仅对于mouseover和mouseout事件）
        if (event.relatedTarget){
            return event.relatedTarget;
        } else if (event.toElement){
            return event.toElement;
        } else if (event.fromElement){
            return event.fromElement;
        } else {
            return null;
        }
    },
    getWheelDelta: function(event) {//获取鼠标滚轮增量值，检测是否包含WheelDelta
        if(event.wheelDelta) {
            return event.wheelDelta;
        }else {
            return -event.detail * 40
        }
    },
    getCharCode: function(event) {//获取键盘按键键码。
        if(typeof event.charCode == 'number') {
            return event.charCode;
        }else {
            return event.keyCode;
        }
    },
    getButton:function(event){//在mouseup或者mousedown的时候，event存在一个button属性，用于判断是按了鼠标左键，右键，还是中键，0鼠标主键按钮，1是中间，2是次键（右键）
        if(document.implementation.hasFeature("MouseEvents","2.0")){
            return event.button;
        }else{
            switch(event.button){
                case 0:
                case 1:
                case 3:
                case 5:
                case 7:
                  return 0;
                case 2:
                case 6:
                   return 2;
                case 4:
                  return 1;

            }
        }

    }
};
```

<!-- more -->

## 使用方法

例如click事件

```js
var btn1 = document.getElementById("myBtn1");
var handler = function(){
alert("hello haorooms");
}

EventUtil.addHandler(btn1, "click", handler);
//EventUtil.removeHandler(btn1, "click", handler);
```