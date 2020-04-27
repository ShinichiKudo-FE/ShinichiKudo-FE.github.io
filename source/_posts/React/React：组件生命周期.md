---
title: React：组件生命周期
date: 2018-04-04 23:47:59
tags: "React"
categories: "React"
---


在组件的整个生命周期中，随着该组件的props或者state发生改变，其DOM表现也会有相应的变化。一个组件就是一个状态机，对于特定地输入，它总返回一致的输出。

一个React组件的生命周期分为三个部分：**实例化、存在期和销毁时**。

# 实例化
当组件在*客户端*被实例化，第一次被创建时，以下方法依次被调用：

1、getDefaultProps
2、getInitialState  

1、2步骤使用Es6语法则为
```javascript
constructor(props){
    super(props);
    this.state = {}
}
```

3、componentWillMount
4、render
5、componentDidMount

当组件在*服务端*被实例化，首次被创建时，以下方法依次被调用：
1、getDefaultProps
2、getInitialState
3、componentWillMount
4、render

componentDidMount 不会在服务端被渲染的过程中调用。

## getDefaultProps
对于每个组件实例来讲，这个方法只会调用一次，该组件类的所有后续应用，getDefaultPops 将不会再被调用，其返回的对象可以用于设置默认的 props(properties的缩写) 值。

```javascript
var Hello = React.createClass({
    getDefaultProps:function(){
        return{
            name : "tom",
            git : "dwqs"
        }
    },
    render : function(){
        return{
            <div>Hello,{this.props.name},git username is {this.props.git}</div>
        }
    }
})
ReactDOM.render(<Hello/>,document.body)
```
<!-- more -->

也可以在挂载组件的时候设置 props：
```javascript
var data = [{title:'Hello'}];
<Hello data={data}/>>
```

或者调用 `setProps` （一般不需要调用）来设置其 props

```javascript
var data = [{title:'data'}];
var Hello = ReactDOM.render(</Demo>,document.body);
Hello.setProps({data:data})
```

**但只能在子组件或组件树上调用 setProps。别调用 this.setProps 或者 直接修改 this.props。将其当做只读数据。**

React通过 `propTypes` 提供了一种验证 props 的方式，propTypes 是一个配置对象，用于定义属性类型：

```javascript
var survey = React.createClass({
    propTypes:{
        survey:React.propTypes.shape({
            id: React.propType.number.isRequered;
        }).isRequiered,
        onClick:React.propTypes.func,
        name: React.propTypes.string,
        score: React.propTypes.array
        //...
    },
   // ...
})
```

组件初始化时，如果传递的属性和 propTypes 不匹配，则会打印一个 console.warn 日志。如果是可选配置，可以去掉.isRequired。propTypes 的详解可[戳此](https://facebook.github.io/react/docs/typechecking-with-proptypes.html)

## getInitalState
对于组件的每个实例来说，这个方法的调用getInitalState有且只有一次，getInitalState用来初始化每个实例的 state，在这个方法里，可以访问组件的 props。每一个React组件都有自己的 state，其与 props 的区别在于 state只存在组件的内部，props 在所有实例中共享。

> getInitialState 和 getDefaultPops 的调用是有区别的，getDefaultPops 是对于组件类来说只调用一次，后续该类的应用都不会被调用，而 getInitialState 是对于每个组件实例来讲都会调用，并且只调一次

```javascript
var LikeButton = React.createClass({
    getInitalState:function(){
        return {liked :false}
    }
    handleClick:function(event){
        this.setState({liked : !this.state.liked})
    }
    render:function(){
        var text = this.state.liked ? 'liked' : 'dont like'
        return(
            <p onClick = {this.handleClick}>
                you {text} this book;
            </p>
        )
    }
})
ReactDOM.render(
  <LikeButton />,
  document.getElementById('example')
);
```
每次修改 state，都会重新渲染组件，实例化后通过 state 更新组件，会依次调用下列方法

1、shouldComponentUpdate
2、conponentWillUpdate
3、render
4、conponentDidUpdate

但是不要直接修改 *this.state*，要通过 *this.setState* 方法来修改。

## componentWillMount
在首次渲染执行前立即调用且仅调用一次。如果在这个方法内部调用 setState 并不会触发重新渲染，这也是在 render 方法调用之前修改 state 的最后一次机会。

## render
该方法会创建一个虚拟DOM，用来表示组件的输出。对于一个组件来讲，render方法是唯一一个必需的方法。render方法需要满足下面几点：

1.只能通过this.state和this.props访问数据（不能修改）
2.可以返回null和false 或者任何React组件
3.只能出现一个顶级组件，不能返回一组元素
4.不能改变组件的状态
5.不能修改Dom的输出

render方法返回的结果并不是真正的DOM元素，而是一个虚拟的表现，类似于一个DOM tree的结构的对象。react之所以效率高，就是这个原因。

## componentDidMount
该方法不会在服务端被渲染的过程中调用。该方法被调用时，已经渲染出真实的 DOM，可以再该方法中通过 `this.getDOMNode()` 访问到真实的 DOM(推荐使用 `ReactDOM.findDOMNode()`)。

```javascript
var data = [..];
var comp = React.createClass({
    render: function(){
        return <imput .. />
    },
    conponentDidMount: function(){
        $(this.getDOMNode()).autoComplete({
            src: data
        })
    }
})
```

由于组件并不是真实的 DOM 节点，而是存在于内存之中的一种数据结构，叫做虚拟 DOM （virtual DOM）。只有当它插入文档以后，才会变成真实的 DOM 。**有时需要从组件获取真实 DOM 的节点，这时就要用到 ref 属性**

```javascript
var Aera = React.createClass ({

    render:function(){
        this.getDOMNode();//render调用时，这里未挂载，会报错
        return(
            return <canvas ref='mainCanvas'>
        )
    }，
    componentDidMount: function(){
        var canvas = this.refs.mainCanvas.getDOMNode();
        //这是有效的，可以访问到 Canvas 节点
    }
})
```

需要注意的是，由于 `this.refs.[refName]` 属性获取的是真实 DOM ，所以必须等到虚拟 DOM 插入文档以后，才能使用这个属性，否则会报错

# 存在期
此时组件已经渲染好并且用户可以与它进行交互，比如鼠标点击，手指点按，或者其它的一些事件，导致应用状态的改变，你将会看到下面的方法依次被调用
1、componentWillReceiveProps
2、shouldComponentUpdate
3、componentWillUpdate
4、render
5、componentDidUpdate

## componentWillReceiveProps
组件的 props 属性可以通过父组件来更改，这时，componentWillReceiveProps 将来被调用。可以在这个方法里更新 state,以触发 render 方法重新渲染组件。
```javascript
componentWillReceliveProps:function(nextProps){
    if(nextProps.click !== undefined){
        this.setState({
            checked: nextProps.checked
        })
    }
}
```
## shouldComponentUpdate
如果你确定组件的 props 或者 state 的改变不需要重新渲染，可以通过在这个方法里通过返回 `false` 来阻止组件的重新渲染，返回 `false `则不会执行 render 以及后面的 `componentWillUpdate`，`componentDidUpdate `方法。

该方法是非必须的，并且大多数情况下没有在开发中使用。
```javascript
shouldComponentUpdate: function(nextProps, nextState){
    return this.state.checked === nextState.checked;
    //return false 则不更新组件
}
```

## componentWillUpdate
这个方法和 componentWillMount 类似，在组件接收到了新的 props 或者 state 即将进行重新渲染前，componentWillUpdate(object nextProps, object nextState) 会被调用，注意**不要在此方面里再去更新 props 或者 state**。

## componentDidUpdate
这个方法和 componentDidMount 类似，在组件重新被渲染之后，componentDidUpdate(object prevProps, object prevState) 会被调用。可以在这里访问并修改 DOM。

# 销毁期

## componentWillUnmount
每当React使用完一个组件，这个组件必须从 DOM 中卸载后被销毁，此时 componentWillUnmout 会被执行，完成所有的清理和销毁工作，在 conponentDidMount 中添加的任务都需要再该方法中撤销，如创建的定时器或事件监听器。

当再次装载组件时，以下方法会被依次调用：
1、getInitialState
2、componentWillMount
3、render
4、componentDidMount

# 反模式
在 getInitialState 方法中，尝试通过 this.props 来创建 state 的做法是一种反模式。

```javascript
//反模式
getDefaultProps: function(){
    return {
        data: new Date()
    }
},
getInitialState: function(){
    return {
        day: this.props.date - new Date()
    }
},
render: function(){
    return <div>Day:{this.state.day}</div>
}
```
经过计算后的值不应该赋给 state，正确的模式应该是在渲染时计算这些值。这样保证了计算后的值永远不会与派生出它的 props 值不同步。
```javascript
getDefaultProps: function(){
    return {
        data: new Date()
    }
},
render: function(){
    var day = this.props.date - new Date();
    return <div>Day:{day}</div>
}
```
如果只是简单的初始化 state，那么应用反模式是没有问题的。

# 总结
下面的一张图总结组件的生命周期：
![组件生命](https://cloud.githubusercontent.com/assets/7871813/17771719/2c764710-6577-11e6-8393-f6acc6f262c2.png)

[原文地址](https://github.com/dwqs/blog/issues/15)