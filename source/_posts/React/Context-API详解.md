---
title: Context API详解
date: 2018-04-03 22:09:00
tags: "React"
categories: "React"
---

React在版本`16.3-alpha`里引入了新的Context API，社区一片期待之声。我们先通过简单的例子，看下新的Context API长啥样，然后再简单探讨下新的API的意义。

[Demo地址](https://github.com/chyingp/blog/tree/master/demo/2018.02.08-react-16.3)

# 看下新的Context API
需要安装16.3-alpha版本的react。
下面，直接来看代码，如果用过`react-redux`应该会觉得很眼熟。
首先，创建`context`实例：

```javascript
import React from 'react';
import ReactDOM from 'react-dom';

// 创建context实例
const ThemeContext = React.createContext({
  background: 'red',
  color: 'white'
});

```
然后，定义App组件，注意这里用到了`Provider`组件，类似`react-redux`的Provider组件

<!-- more -->

```javascript
class App extends React.Component {

  render () {
    return (
      <ThemeContext.Provider value={{background: 'green', color: 'white'}}>
        <Header />
       </ThemeContext.Provider>
    );
  }
}
```

接下来，定义`Header`、`Title`组件。注意：

Title组件用到了`Consumer`组件，表示要消费`Provider`传递的数据。
Title组件是App的孙组件，但跳过了`Header`消费数据。

```javascript
class Header extends React.Component {
  render () {
    return (
      <Title>Hello React Context API</Title>
    );
  }
}

class Title extends React.Component {
  render () {
    return (
      <ThemeContext.Consumer>
        {context => (
          <h1 style={{background: context.background, color: context.color}}>
            {this.props.children}
          </h1>
        )}
      </ThemeContext.Consumer>
    );
  }
}
```
最后常规操作
```javascript
ReactDOM.render(
  <App />, 
  document.getElementById('container')
);
```

# 为什么有新的Context API
过`redux + react-redux`的同学，应该会觉得新的Context API很眼熟。而有看过`react-redux`源码的同学就知道，react-redux本身就是基于旧版本的Context API实现的。

既然已经有了现成的解决方案，为什么还会有新的Context API呢？

> 现有Context API的实现存在一定问题：比如当父组件的shouldComponentUpdate性能优化，可能会导致消费了context数据的子组件不更新。
1.降低复杂度：类似redux全家桶这样的解决方案，给项目引入了一定的复杂度，尤其是对方案了解不足的同学，遇到问题可能一筹莫展。
2.新Context API的引入，一定程度上可以不少项目对redux全家桶的依赖。

[原文地址](http://cnodejs.org/topic/5a7bd5c4497a08f571384f03)