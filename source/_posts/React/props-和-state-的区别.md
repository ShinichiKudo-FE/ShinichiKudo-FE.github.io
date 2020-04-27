---
title: 'props 和 state 的区别 '
date: 2018-04-08 11:22:57
tags: "React"
categories : "React"
---

react是基于状态实现对DOM控制和渲染。组件状态可分为两种：一种是组件间的状态传递，另一种是组件的内部状态。这两种状态使用`props`和`state`表示。`props`用于父组件向子组件的数据传递，组件内部也有自己的状态：`state`，这些状态只能在组件内部修改。

# 数据流与props

react的数据流是单向的，只会从父组件传递到子组件。属性props（properties）是父子组件进行传递状态的接口。React会向下遍历整个组件树，并重新渲染使用这个属性的组件。

## 设置props

可以在组件挂载时设置`props`

```javascript
var sites = [{title:'baidu.com'}];
<ListSites sites = {sites}/>
```

也可以通过调用组件实例的`SetProps()`方法设置props

```javascript
var sites = [{title:'baidu.com'}];
ReactDOM.render(
    <ListSites/>,
    document.getElementById('example')
)
```

<!-- more -->

SetProps()方法只能用于组件外部调用，不能在组件内使用`this.SetProps()`修改组件属性。组件内部的this.props是只读的，只能用于访问props属性，不能修改组件自身的属性。

### JSX语法中的属性设置

JSX语法中，props可以设置为字符串：

```javascript
<a href="http://itbilu.com">IT笔录</a>
```

或是通过{}语法设置：

```javascript
var obj = {url:'itbilu.com', name:'IT笔录'};
<a href="http://{obj.url}">{obj.name}</a>
```

JSX还支持将props直接设置为一个对象：

```javascript
class Site extends React.Component{
    render(){
        var obj = ({url:'baidu.com',name:'IT笔录'})
        return(
            <Site {...obj}/>
        )
    }
}
```

props还可以用来添加事件处理：

```javascript
var saveBtn = React.createClass({
    render:function(){
        <a onClick={this.handleClick}>保存</a>
    }
    handleClick:function(){
        //...
    }
})
```

### props的传递

组件接收上级组件的props，并传递到下级组件，如：

```javascript
var myCheackBox = React.createClass ({
    render:myCheackBox(){
        var myClass = this.props.cheacked ? 'MyCheacked':'MyCheackBox';
        return(
            <div className = {myClass} onClick = {this.props.onClick}>
                {this.props.children}
            </div>
        )
    }
})
React.render(
    <MyCheackBox cheacked = {true} onClick = {console.log.bind(console)}>
        hello world
    <MyCheackBox/>,
    document.getElementById('example')
)
```

## 组件内部状态与State

props可以理解为父子组件的状态传递，而React有自己内部的状态，这个内部状态则用State表示。

如，用state定义一个<DropDown />组件的状态：

```javascript
var SiteDropdown = React.createClass({
     getInitalState: function() {
        return: {
        showOptions: false
        }
     }
     render:function(){
         var opts;
         if(this.state.showOptions){
             <ul>
                <li>itbilu.com</li>
                <li>yijiebuyi.com</li>
                <li>niefengjun.cn</li>
             </ul>
         };
         return( <div onClick={this.handleClick} >
                </div>
        )
     },
     handleClick:function(){
         this.SetState ({
             showOptions: true
         })
     }
  }
})
```

随着state的变化，render也会调用，react会对比render返回的值，如果有变化就会DOM；
state与props类似，只能通过setState()方法修改。不同的是，state只能在组件内部使用，其实是对组件本身状态的一个引用


## props与state的比较

`React`会根据`props`和`state`更新视图状态，虽然二者有相似之处，但是两者的应用范围不同。具体表现如下：

> * props会在整个组件数中传递数据和配置，props可以设置任命类型的数据，应该把当作组件的数据源。其不但可以用于上级组件与下级组件的通信，也可以用作事件处理器

> * state只能组件内部使用，state只应该用于存储简单的视图状（如：上面示例用于控制下拉框的可见状态）

> * props与state都不能直接修改，而应该分别使用SetProps()和SetProps()修改



[原文地址](https://itbilu.com/javascript/react/4k5RfzDKx.html#props)

