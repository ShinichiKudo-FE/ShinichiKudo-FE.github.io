---
title: redux学习之开店卖水果
date: 2018-04-25 10:34:34
tags: "Redux"
categories: "Redux"
---
# Redux 是什么

每当我们谈及到 [redux](https://github.com/reactjs/redux)，大家都会说是 [react](https://github.com/facebook/react) 的状态管理工具。这么说确实没错，毕竟 redux 项目也是 `React Community` 组织下的一个子项目。而且 redux 的诞生也是和 react 这个 ui 库急需一个状态管理解决方案有很大的联系。但是 redux 和 react 并没有任何的耦合。虽然它们经常一起用，但是 `redux 的用途并不局限于 react`，或者说，和 `react 的结合只是 redux 的使用方式之一`。

那么撇开 react 不谈， redux 到底是什么呢？我们看一下这个例子。

在实际的开发当中，我们可能会碰到这样的需求：监听一个事件，当事件触发的时候，我们可以做一些想做的事情。

## redux基础用法---卖水果

一天，程序员阿大（化名）想要去买水果吃，发现小区周围居然没有水果店，于是就打算自己开一个水果店赚点小钱。
阿大分析了一下水果店的营业模式。其实就是**处理每一位顾客的需求**，然后记账看看每天的盈亏。那么抽象成程序就是`监听顾客的行为，并把每个行为的结果都记在账上`，这正好是 **redux** 所擅长的。阿大胸有成竹，说着就开始写了起来：

### 首先模拟顾客的购买行为：

```js
const action = {
  type: 'BUY_APPLE', // 买苹果
  payload: {
    count: 2 // 买 2 斤
  }
}
```

那不同的顾客要的斤数可能不同，于是他写了下面这个方法：

```js
/**
 * 只要知道斤数就可以快速生成顾客的各种需求
 * @param {number} num 顾客要买的斤数
 */
function buyApple(num) {
  return {
    type: 'BUY_APPLE',
    payload: {
      count: num
    }
  }
}
```

然后是账本的结构，记录每天卖了多少斤：
<!-- more -->

```js
// 被托管的数据 state
// 账本，今天已卖的苹果：0 斤；为了简便，就只举一个例子，事实上还有很多其他水果，大家自行脑补
const state = { apple: 0 };
```

好了，现在顾客需求，账本都有了，那谁来记账呢？所以阿大请了一个`收银员`负责记账，并告诉他这么记账：

```js
/**
 * 监听函数 listener
 * 收银员只要知道顾客的需求就能正确的操作账本
 * @param {object} state 账本
 * @param {object} action 顾客的需求
 */
function reducer(state, action) {

  // 注册 ‘买苹果’ 事件
  // 如果有人买了苹果，加上顾客买的斤数，更新账本
  if (action.type === 'BUY_APPLE') {
    return Object.assign({}, state, {
      apple: state.apple + action.payload.count
    });
  }

  // 没注册的事件不作处理
  // 买咱们店里没有的东西，不更新账本，原样返回
  return state;
}
```

好，万事俱备，可以正式的监听顾客的购买需求并更新账本了：

```js
const {createStore} = require('redux');

// 创建水果店需要收银员（监听函数 listener）和账本（被托管的数据）
const store = createStore(reducer, state);
```

不仅如此，redux 还提供了一个功能，每服务一个顾客，都可以额外做一些事情，于是阿大就想看看每笔交易之后的账本：

```js
// store.getState() 可以获取最新的 state
store.subscribe(() => console.log(JSON.stringify(store.getState())));
```

好了，顾客开始来买水果了：

```js
// 触发用户购买水果的事件
// 销售员开始销售
store.dispatch(buyApple(3)); // {"apple":3}
store.dispatch(buyApple(4)); // {"apple":7}
```

店铺稳定的运营了下去，阿大心里美滋滋~

### 讲解

不过在此之前要先说 redux 特别讲究也是特别重要的 3 点：

* 只能有唯一的 store 对象保存整个应用的 state
* state 是只读的，只能通过 dispatch(action) 的方式来改变 state
* reducer 必须是纯函数

#### action

action 是行为信息的抽象，对象类型，它描述发生了什么。这个对象必须有一个 type 属性，对于对象里面的其他内容，redux 不做限制。但是推荐符合 Flux Standard Action 规范：

```js
{
  type: 'ACTION_TYPE',
  payload, // action 携带的数据
}
```

#### action creator

action creator 顾名思义就是用来创建 action 的，action creator 只简单的返回 action。

```js
function createAction(num) {
  return {
    type: 'ACTION_TYPE',
    payload,
  }
}
```

#### state

state 是被托管的数据，也就是每次触发监听事件，我们要操作的数据。

#### reducer

reducer 是用来控制 state 改变的函数。action 描述了发生了什么，但是并不会知道相应的 state 该怎么改变。对于不同的 action，相应的 state 变化是用 reducer 来描述的。

reducer 接受两个函数，第一个是 `state`，第二个是 `action`，并返回计算之后新的 state。reducer 必须是一个**纯函数**，对于相同的输入 state 和 action，一定会返回相同的新的 state。

```js
nextState = reducer(prevState, action);
```

因为 reducer 是纯函数，所以原来的 **prevState** 并不会改变，新的 **nextState** 是一个最新的快照。

#### store

store 是把上面三个元素合起来的一个大对象:

```js
{
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}
```

> 它负责：
* 托管应用的 state
* 允许通过 store.getState() 方法访问到托管的 state
* 允许通过 store.dispatch() 方法来触发 action 更新 state
* 允许通过 store.subscribe() 注册监听函数监听每一次的 action 触发
* 允许注销通过 store.subscribe() 方法注册的监听函数

```js
// 注册
const unsubscribe = store.subscribe(() => { /** do something */});

// 注销
unsubscribe();
```

![图解](https://user-gold-cdn.xitu.io/2018/4/18/162d7d65eb48257f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## reducer拆分

谁知道水果店生意越来越好，于是阿大开始拓展业务，不仅卖水果，还卖起了生鲜，于是有了水果部和生鲜部。

于是阿大想了想未来购买生鲜的顾客的行为：

```js
// 买生鲜 - 鸡蛋
function buyEgg(num) {
  return {
    type: 'BUY_EGG',
    payload: {
      count: num
    }
  }
}
```

分了不同的部门之后，不同的业务有不同的记账方式，得分账记了，开来要增加一个生鲜的记账本：

```js
const freshState = {
  egg: 0
};
```

原来的水果账本也要改个名字：

```js
//- const state = {
+ const fruitsState = {
    apple: 0
  };
```

然后增加生鲜部的收银员, 管理生鲜账本 `freshState`：

```js
// 生鲜部收银员
function freshReducer(state = freshState, action) {
  if (action.type === 'BUY_EGG') {
    return Object.assign({}, state, {
      egg: state.egg + action.payload.count
    });
  }
  return state;
}
```

然后原来水果部的收银员管理水果账本 fruitsState 需要修改下：

```js
// 水果部收银员
//- function reducer(state, action) {
+ function fruitReducer(state = fruitState, action) {
   if (action.type === 'BUY_APPLE') {
     return Object.assign({}, state, {
       apple: state.apple + action.payload.count
     });
   }
   return state;
 }
```

但是阿大并不想看各个部门的分账本，他只想看一个总账本就好了。刚好 redux 提供了 **combineReducers** 功能，可以把各个收银员管理的账本合起来：

```js
//- const { createStore } = require('redux');
+ const { createStore, combineReducers } = require('redux');

// 总账本
+ const state = {
+   fruits: fruitsReducer,
+   fresh: freshReducer
+ };

// 总收银员
+ const reducer = combineReducers(state);

// 创建新的水果生鲜店
//- const store = createStore(reducer, state);
+ const store = createStore(reducer);

```

这样，水果生鲜店就可以营业了，销售员又开始处理顾客的购物需求了：

```js
store.dispatch(buyApple(3)); // {"fruit":{"apple":3},"fresh":{"egg":0}}
store.dispatch(buyEgg(1)); // {"fruit":{"apple":3},"fresh":{"egg":1}}
store.dispatch(buyApple(4)); // {"fruit":{"apple":7},"fresh":{"egg":1}}
store.dispatch(buyEgg(8)); // {"fruit":{"apple":7},"fresh":{"egg":9}}
// ...
```

### 图解：

![reducer](https://user-gold-cdn.xitu.io/2018/4/18/162d7f51769b18c2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### combineReducers

当业务场景越来越复杂的时候，state 的结构也会变得越来越复杂而且庞大。如果只用一个 reducer 来计算 state 的变化势必会特别麻烦。这个时候我们就可以把 state 里独立的数据分离出来，单独用一个 reducer 来计算，然后再通过 **combineReducers** 方法合入到 state 中。
combineReducers 接收一个对象，这个对象就是最终的 state

```js
const reducer = combineReducers({
  fruits: fruitsReducer,
  fresh: freshReducer
});
```

## 处理 async action

阿大通过 redux 的 bindReducers 方法将水果店的业务分治成功，店铺也越做越大。以至于有顾客开始想要买一些进口的水果生鲜。

阿大考虑了一下，决定继续拓展这个店铺，从事进口商品的销售。首先是顾客的需求行为需要购买进口水果生鲜：

```js

// 买水果 - 进口苹果
+ function buyImportedApple(num) {
+   return {
+     type: 'BUY_IMPORTED_APPLE',
+     payload: {
+       num
+     }
+   }
+ }

// 买生鲜 - 进口鸡蛋
+ function buyImportedEgg(num) {
+   return  {
+     type: 'BUY_IMPORTED_EGG',
+     payload: {
+       num
+     }
+   }
+ }

```

然后水果部和生鲜部的账本也要更新啦：

```js
// 水果账本
 const fruitState = {
   orange: 0,
   apple: 0,
   banana: 0,
+  importedApple: 0
 };

 // 生鲜账本
 const freshState = {
   egg: 0,
   fish: 0,
   vegetable: 0,
+  importedEgg: 0
 };

```

同样的，相应部门的收银员们也要学会怎么处理进口水果生鲜的记账，他们的记账方式要改成下面这样：

```js
// 水果部收银员
function fruitReducer(state = fruitState, action) {

  // 如果有人买了相应的水果，更新账本
  switch (action.type) {
    case 'BUY_APPLE':
      return Object.assign({}, state, {
        apple: state.apple + action.payload.count
      });
    case 'BUY_IMPORTED_APPLE':
      return Object.assign({}, state, {
        importedApple: state.importedApple + action.payload.count
      });

    // 买其他的东西，不更新账本，原样返回
    default: return state;
  } ;
}

// 生鲜部收银员
function freshReducer(state = freshState, action) {
  switch (action.type) {
    case 'BUY_EGG':
      return Object.assign({}, state, {
        egg: state.egg + action.payload.count
      });
    case 'BUY_IMPORTED_EGG':
      return Object.assign({}, state, {
        importedEgg: state.importedEgg + action.payload.count
      });
    default: return state;
  } ;
}
```

可是这时候阿大发现，进口水果生鲜不能大量存在自己仓库卖，因为它们又贵又容易坏，只有当顾客需要买的时候，才能去采购这些水果生鲜，于是阿大又雇了一个**采购员**专门负责处理要买进口水果和生鲜的顾客，等到货了再通知销售员取货给顾客：

```js
// 采购商品生成器，不同的商品需要不同的时间采购
function fetchGoodsGenerator(time, action) {

  // 用延时模拟采购时间
  const timer = setTimeout(() => {
    clearTimeout(timer);

    // 采购完成，通知销售员
    store.dispatch(action);
  }, time);
}

// 采购进口苹果需要 2 天（2s）
function fetchImportedApple(action) {
  fetchGoodsGenerator(2000, action);
}

// 采购进口鸡蛋需要 3 天（3s）
function fetchImportedEgg(action) {
  fetchGoodsGenerator(3000, action);
}

// 采购员
const API = {
  fetchImportedApple, // 采购进口苹果
  fetchImportedEgg // 采购进口鸡蛋
}
```

好了，布置完了之后，顾客开始来买水果生鲜了：

```js
// 销售员开始销售，采购员开始采购
store.dispatch(buyApple(3));
API.fetchImportedApple(buyImportedApple(10));
store.dispatch(buyEgg(1));
API.fetchImportedEgg(buyImportedEgg(10));
store.dispatch(buyApple(4));
API.fetchImportedApple(buyImportedApple(10));
store.dispatch(buyEgg(8));
API.fetchImportedEgg(buyImportedEgg(10));
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

![async action](https://user-gold-cdn.xitu.io/2018/4/18/162d925e3fdc050d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

在实际的开发当中我们经常会调用一些 API 接口获取数据更新 state。刚开始使用 redux 的一个误区就是在 reducer 里接收到异步的 action 之后，就在 reducer 里做异步操作，调用 API。但是这样是错误的。reducer 只能是纯函数，不能有任何副作用。这样才能保证对于相同的输入，一定会有相同的输出。

[原文地址](https://juejin.im/user/57d8a928d203090069c13a21/likes)