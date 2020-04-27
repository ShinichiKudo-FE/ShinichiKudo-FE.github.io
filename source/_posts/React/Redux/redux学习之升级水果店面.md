---
title: redux学习之升级水果店面
date: 2018-04-25 13:59:29
tags: "Redux"
categories: "Redux"
---

## 优雅的处理 async action

阿大通过请了一个采购员完成了耗时的进口商品的售卖。

但是，阿大同时也发现了一个问题：顾客要买水果生鲜的话需要找**销售员**，要买进口水果生鲜的话要找**采购员**，这样的话，顾客需要找不同的人，很麻烦。阿大想了想，能不能让顾客只找**销售员**，然后**销售员**如果有需求再找**采购员**采购呢。

阿大想到了办法，让**销售员**把所有的需求都告诉采购员，然后**采购员**再传递给**收银员**，这样，如果是需要采购的水果生鲜，就可以独自去处理了，这样就需要把**采购员**改成这样了：

```js
const API = store => {
  // 和 收银员 对接的方式
  const next = store.dispatch;
  // 接管销售员传来的顾客需求
  store.dispatch = action => {

    // 处理完了之后再对接 收银员
    switch (action.type) {
      case 'BUY_IMPORTED_APPLE': fetching(2000, () => next(action)); break;
      case 'BUY_IMPORTED_EGG': fetching(3000, () => next(action)); break;
      default: next(action);
    }
  }
}

API(store);
```

然后顾客来了之后就都只用找销售员了：
<!-- more -->

```js
store.dispatch(buyApple(3));
store.dispatch(buyImportedApple(10));
store.dispatch(buyEgg(1));
store.dispatch(buyImportedEgg(10));
store.dispatch(buyApple(4));
store.dispatch(buyImportedApple(10));
store.dispatch(buyEgg(8));
store.dispatch(buyImportedEgg(10));
// {"fruit":{"apple":3,"importedApple":0},"fresh":{"egg":0,"importedEgg":0}}
// {"fruit":{"apple":3,"importedApple":0},"fresh":{"egg":1,"importedEgg":0}}
// {"fruit":{"apple":7,"importedApple":0},"fresh":{"egg":1,"importedEgg":0}}
// {"fruit":{"apple":7,"importedApple":0},"fresh":{"egg":9,"importedEgg":0}}
// {"fruit":{"apple":7,"importedApple":10},"fresh":{"egg":9,"importedEgg":0}}
// {"fruit":{"apple":7,"importedApple":20},"fresh":{"egg":9,"importedEgg":0}}
// {"fruit":{"apple":7,"importedApple":20},"fresh":{"egg":9,"importedEgg":10}}
// {"fruit":{"apple":7,"importedApple":20},"fresh":{"egg":9,"importedEgg":20}}

```

### 图解

![asyncAction](https://user-gold-cdn.xitu.io/2018/4/20/162e3327a35fb0a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 编写和使用中间件

阿大通过改善流程对接完成了水果店的升级。

但是阿大又有一个新的想法，他想详细的看看每一个顾客的`购买需求`来了之后，账本的前后变化。看来又要加一个新角色**记录员**了。难道要像加采购员那样手动的一个个的加吗？那可太麻烦了。正好阿大发现 redux 里有一个功能就是**中间件**。中间件是干嘛的呢？简而言之，就是把顾客的需求从销售员到收银员之间加上各种角色来处理。每一个角色就是一个中间件。接下来阿大就开始来写中间件了。

redux 中间件写起来其实很简单，就是一个函数而已。按照它的要求。这个函数接受一个**{ dispatch, getState }**对象作为参数，然后返回一个 **action**

那这样，就可以把原来的`采购员也改造成中间件`了，其实采购员就是拿到了顾客需求之后让顾客的需求延迟 **dispatch**，这用`延迟用函数`就可以做到了:

```js
// next 是用中间件增强之后的 dispatch
// dispatch 是最原始的 store.dispatch
const thunkMiddleware = ({ dispatch }) => next => action => {
  if (typeof action === 'function') {

    // 函数形式的 action 就把 dispatch 给这个 action，让 action 决定什么时候 dispatch （控制反转）
    return action(dispatch);
  }

  // 普通的 action 就直接传递给下一个中间件处理
  return next(action);
}
```

然后我们就需要把原来的顾客需求改一下了：

```js
// 买水果 - 进口苹果
function buyImportedApple(num) {

  // 返回一个函数类型的 action，这个函数接受 dispatch，可以决定什么时候 dispatch
  return dispatch => API.fetchImportedApple(() => dispatch({
    type: 'BUY_IMPORTED_APPLE',
    payload: num
  }));
}

// 买生鲜 - 进口鸡蛋
function buyImportedEgg(num) {
  return dispatch => API.fetchImportedEgg(() => dispatch({
    type: 'BUY_IMPORTED_EGG',
    payload: num
  }));
}
```

然后`采购员`就可以只负责采购了，改回去：

```js
// 采购商品生成器，不同的商品需要不同的时间采购
function fetching(time, callback) {

  // 用延时模拟采购时间
  const timer = setTimeout(() => {
    clearTimeout(timer);

    // 采购完成，通知销售员
    callback();
  }, time);
}

// 采购进口苹果需要 2 天（2s）
function fetchImportedApple(callback) {
  fetching(2000, callback);
}

// 采购进口苹果需要 3 天（3s）
function fetchImportedEgg(callback) {
  fetching(3000, callback);
}

// 采购员
const API = {
  fetchImportedApple, // 采购进口苹果
  fetchImportedEgg // 采购进口鸡蛋
}
```

然后，我们在添加一个`记录员`的中间件：

```js
const loggerMiddleware = ({ getState }) => next => action => {
  console.log(`state before: ${JSON.stringify(getState())}`);
  console.log(`action: ${JSON.stringify(action)}`);
  const result = next(action);
  console.log(`state after: ${JSON.stringify(getState())}`);
  console.log('================================================');
  return result;
}
```

删除掉原来的监听：

```js
//- store.subscribe(() => console.log(JSON.stringify(store.getState())));
```

好了，接下来就可以通过 redux 的 **applyMiddleware** 来串联起这些中间件啦。

```js
const { createStore, combineReducers, applyMiddleware } = require('redux');

// 中间件的调用顺序是从右到左
const store = createStore(reducer, applyMiddleware(thunkMiddleware, loggerMiddleware));
```

好了，大功告成，开始服务顾客：

```js
store.dispatch(buyApple(3));
store.dispatch(buyImportedApple(10));

// state before: {"fruit":{"apple":0,"importedApple":0},"fresh":{"egg":0,"importedEgg":0}}
// action: {"type":"BUY_APPLE","payload":3}
// state after: {"fruit":{"apple":3,"importedApple":0},"fresh":{"egg":0,"importedEgg":0}}
// ================================================
// state before: {"fruit":{"apple":3,"importedApple":0},"fresh":{"egg":0,"importedEgg":0}}
// action: {"type":"BUY_EGG","payload":1}
// state after: {"fruit":{"apple":3,"importedApple":0},"fresh":{"egg":1,"importedEgg":0}}
// ================================================

```

上面我们写的两个中间件其实就是 `redux-thunk` 和 `redux-logger` 的简版。在实际中，推荐使用它们，会更可信。

### 图解

![redux-chunk](https://user-gold-cdn.xitu.io/2018/4/23/162f0a03fc97dbca?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

编写 redux 中间件需要按照要求来，返回这样的函数

```js
// 中间件接受一个对象，里面有原始的 dispatch，和 getState 方法用于获取 state
// 中间件函数返回一个函数，这个函数接受一个 next 参数，这个 next 是下一个中间件要做的事情 action => { ... }
function thunkMiddleware({ dispatch, getState }) {
  return function(next) {
    return function(action) {
      // 做你的事情
    }
  }
}
```

[原文地址](https://juejin.im/user/57d8a928d203090069c13a21/likes)