---
title: 重学React(三)
tags: ''
categories: ''
date: 2020-08-11 17:50:07
---

# 前言

终于到了期盼已久的HOOK环节了，我最初学react的时候是版本为15.4.2(特地去看了react仓库看了[更新日志](https://github.com/facebook/react/blob/master/CHANGELOG.md#1562-september-25-2017))算是很早的学习，可惜自己学的东西在工作中实践不了，当时也刚参加工作，又推动不了新技术的落地。无奈过段时间就忘了。计算机技术只有不断敲，看了用不到7天内就会忘得差不多了。当时看到class组件的时候觉得帅呆了，因为那时es6正当时，看到很多人都是使用jsx语法，class声明组件等等，觉得自己赶上了好时机。后来情况就是用不到放弃了，虽然用不到但是平时关注一些新的技术，react HOOK 发布时，也看了一些文章，一些大佬也推荐使用hook的语法写react,到了现在版本16.13.1，我才开始真正接触HOOK

根据官网的介绍：

<Big>**Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性**<Big>。

> Hook的三个特性

* 完全可选的
* 100%向后兼容的
* 现在就用 

## 使用Hook的动机
[具体介绍](https://react.docschina.org/docs/hooks-intro.html#motivation)
1.在组件之间复用状态逻辑很难

2.复杂组件变得难以理解

3.难以理解的 class


## 使用State Hook

```js
// demo

import React, { useState } from 'react';

function Example() {
  // 声明一个叫 "count" 的 state 变量
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
};

// 等价class组件

class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}

```

在函数组件中，我们没有 this，所以我们不能分配或读取 this.state。

## 使用Effect Hook

`Effect Hook` 可以让你在函数组件中执行副作用操作

```js
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // Similar to componentDidMount and componentDidUpdate:
  useEffect(() => {
    // Update the document title using the browser API
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

*有些副作用可能需要清除，所以需要返回一个函数：*
```js
useEffect(() => {
  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
  };
});
```

*其他的 effect 可能不必清除，所以不需要返回。*
```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
});
```

## Hook的规则

1.只在最顶层使用 Hook

**不要在循环，条件或嵌套函数中调用 Hook**， 确保总是在你的 React 函数的最顶层调用他们。

2.只在 React 函数中调用 Hook

**不要在普通的 JavaScript 函数中调用 Hook**

可以在：
* 在 React 的函数组件中调用 Hook
* 在自定义 Hook 中调用其他 Hook

## FAQ

[更多](https://react.docschina.org/docs/hooks-faq.html#which-versions-of-react-include-hooks)

为什么要在 effect 中返回一个函数？
**这是 effect 可选的清除机制**。每个 effect 都可以返回一个清除函数。如此可以将添加和移除订阅的逻辑放在一起。它们都属于 effect 的一部分。

React 何时清除 effect？ 
**React 会在组件卸载的时候执行清除操作**。正如之前学到的，effect 在每次渲染的时候都会执行。
