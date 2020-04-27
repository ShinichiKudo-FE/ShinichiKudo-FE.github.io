---
title: 助你理解React高阶组件
date: 2018-03-21 17:01:26
tags: "React"
categories: "React"
---
# 高阶组件
高阶组件（HOC）是react中对组件逻辑进行重用的高级技术。但高阶组件本身并不是React API。它只是一种模式，这种模式是由react自身的组合性质必然产生的。

具体而言，**高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件**

高阶组件在React第三方库中很常见，比如Redux的[connect](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)方法和Relay的[createContainer]().

## 使用高阶组件（HOC）解决交叉问题

## 不要改变原始组件，使用组合

## 约定：将不相关的props属性传递给包裹组件

## 约定：最大化使用组合

## 约定：包装显示名字以便于调试

<!-- more -->
# 函数模拟高阶组件
我们通过普通函数来理解什么是高阶组件哦~

1.最普通的方法，一个welcome，一个goodbye。两个函数先从localStorage读取了username，然后对username做了一些处理。
```javascript
    function welcome(){
        let username = localStorage.getItem('username');
        console.log('welcome ' + username);
    }
    function goodbey() {
    let username = localStorage.getItem('username');
    console.log('goodbey ' + username);
    }

    welcome();
    goodbey();
```
2.我们发现两个函数有一句代码是一样的，这叫冗余唉。不好不好~（你可以把那一句代码理解成平时的一大堆代码）

我们要写一个中间函数，读取username,他来负责把username传递给两个函数。
```javascript
function welcome(username) {
    console.log('welcome ' + username);
}

function goodbey(username) {
    console.log('goodbey ' + username);
}

function wrapWithUsername(wrappedFunc) {
    let newFunc = () => {
        let username = localStorage.getItem('username');
        wrappedFunc(username);
    };
    return newFunc;
}

welcome = wrapWithUsername(welcome);
goodbey = wrapWithUsername(goodbey);

welcome();
goodbey();
```

好了，我们里面的wrapWithUsername函数就是一个“高阶函数”。
他做了什么？他帮我们处理了username，传递给目标函数。我们调用最终的函数welcome的时候，根本不用关心username是怎么来的。
我们增加个用户study函数。

```javascript
 function study(username){
     console.log('study' + username);
 }
 study = wrapWithUsername(study);

study();
```

这里你是不是理解了为什么说wrapWithUsername是高阶函数？我们只需要知道，用wrapWithUsername包装我们的study函数后，study函数第一个参数是username。

我们写平时写代码的时候，不用关心wrapWithUsername内部是如何实现的。

# 高阶组件就是一个没有副作用的纯函数。
我们把上一节的函数统统改成react组件。

最普通的组件哦。
welcome函数转为react组件。
```javascript
import React, {Component} from 'react'

class Welcome extends Component {
    constructor(props) {
        super(props);
        this.state = {
            username: ''
        }
    }

    componentWillMount() {
        let username = localStorage.getItem('username');
        this.setState({
            username: username
        })
    }

    render() {
        return (
            <div>welcome {this.state.username}</div>
        )
    }
}

export default Welcome;
```
2.goodbey函数转为react组件。
```javascript
import React, {Component} from 'react'

class Goodbye extends Component {
    constructor(props) {
        super(props);
        this.state = {
            username: ''
        }
    }

    componentWillMount() {
        let username = localStorage.getItem('username');
        this.setState({
            username: username
        })
    }

    render() {
        return (
            <div>goodbye {this.state.username}</div>
        )
    }
}

export default Goodbye;
```
3.现在你是不是更能看到问题所在了？两个组件大部分代码都是重复的唉。
按照上一节wrapWithUsername函数的思路，我们来写一个高阶组件(高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件)。
```javascript
import React, {Component} from 'react'

export default (WrappedComponent) => {
    class NewComponent extends Component {
        constructor() {
            super();
            this.state = {
                username: ''
            }
        }

        componentWillMount() {
            let username = localStorage.getItem('username');
            this.setState({
                username: username
            })
        }

        render() {
            return <WrappedComponent username={this.state.username}/>
        }
    }

    return NewComponent
}

```

这样我们就能简化Welcome组件和Goodbye组件。
```javascript
//welcome
import React, {Component} from 'react';
import wrapWithUsername from 'wrapWithUsername';

class Welcome extends Component {

    render() {
        return (
            <div>welcome {this.props.username}</div>
        )
    }
}

Welcome = wrapWithUsername(Welcome);

export default Welcome;

//goodbye
import React, {Component} from 'react';
import wrapWithUsername from 'wrapWithUsername';

class Goodbye extends Component {

    render() {
        return (
            <div>goodbye {this.props.username}</div>
        )
    }
}

Goodbye = wrapWithUsername(Goodbye);

export default Goodbye;

```
看到没有，高阶组件就是把username通过props传递给目标组件了。目标组件只管从props里面拿来用就好了。

到这里位置，高阶组件就讲完了。你再返回去理解下定义，是不是豁然开朗

>你现在理解react-redux的*connect函数*~
把redux的state和action创建函数，通过props注入给了Component。
你在目标组件Component里面可以直接用this.props去调用redux state和action创建函数了。

```javascript
ConnectedComment = connect(mapStateToProps, mapDispatchToProps)(Component);
```

相当于这样

```javascript
// connect是一个返回函数的函数（就是个高阶函数）
const enhance = connect(mapStateToProps, mapDispatchToProps);
// 返回的函数就是一个高阶组件，该高阶组件返回一个与Redux store
// 关联起来的新组件
const ConnectedComment = enhance(Component);
```

antd的Form也是一样的

const WrappedNormalLoginForm = Form.create()(NormalLoginForm);
# 注意事项
## 不要再render函数中使用高阶组件
React使用的差异算法（称为协调）使用组件标识确定是否更新现有的子对象树或丢掉现有的子树并重新挂载。如果render函数返回的组件和之前render函数返回的组件是相同的，React就递归的比较新子对象树和旧子对象树的差异，并更新旧子对象树。如果他们不相等，就会完全卸载掉旧的之对象树。

## 必须将静态方法做拷贝
有时，给组件定义静态方法是十分有用的。例如，Relay的容器就开放了一个静态方法 getFragment便于组合GraphQL的代码片段。

## Refs属性不能传递
一般来说，高阶组件可以传递所有的props属性给包裹的组件，但是不能传递refs引用。因为并不是像key一样，refs是一个伪属性，React对它进行了特殊处理。如果你向一个由高阶组件创建的组件的元素添加ref应用，那么ref指向的是最外层容器组件实例的，而不是包裹组件。

[原文地址](https://github.com/brickspert/blog/issues/2)