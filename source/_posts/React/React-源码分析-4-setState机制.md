---
title: React 源码分析 4 setState机制
date: 2018-03-27 09:59:56
tags: "React"
categories: "React"
---

# 概述
React作为一门前端框架，虽然只是focus在MVVM中的View部分，但还是实现了View和model的绑定。修改数据的同时，可以实现View的刷新。这大大简化了我们的逻辑，只用关心数据流的变化，同时减少了代码量，使得后期维护也更加方便。这个特性则要归功于setState()方法。React中利用队列机制来管理state，避免了很多重复的View刷新。下面我们来从源码角度探寻下setState机制。

# setState和replaceState
我们都知道setState是以修改和新增的方式改变state的，不会改变没有涉及到的state。而replaceState则用新的state完全替换掉老state。比如

```javascript
this.state = {
  title: "example",
  desc: "this is an example for react"
};

setState({
  title: "new example"
});
console.log("setState: " + JSON.stringify(this.state));    // 1 

replaceState({
  title: "new example"
})
console.log("replaceState: " + JSON.stringify(this.state));    // 2

```
打印如下:
```javascript
setState: {"title":"new example","desc":"this is an example for react"}
replaceState: {"title":"new example"}
```
<!-- more -->
可见，setState不会影响没有涉及到的state，而replaceState则是完完全全的替换。下面让我们进入源码来探寻究竟吧。

## setState
setState方法入口如下
```javascript

ReactComponent.prototype.setState = function (partialState, callback) {
  // 将setState事务放入队列中
  this.updater.enqueueSetState(this, partialState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'setState');
  }
};
```

取名为partialState，有部分state的含义，可见只是影响涉及到的state，不会伤及无辜。enqueueSetState是state队列管理的入口方法，比较重要，我们之后再接着分析。

## replaceState
```javascript
replaceState: function (newState, callback) {
  this.updater.enqueueReplaceState(this, newState);
  if (callback) {
    this.updater.enqueueCallback(this, callback, 'replaceState');
  }
},
```

replaceState中取名为newState，有完全替换的含义。同样也是以队列的形式来管理的。

# equeueSetState
```javascript
enqueueSetState: function (publicInstance, partialState) {
    // 先获取ReactComponent组件对象
    var internalInstance = getInternalInstanceReadyForUpdate(publicInstance, 'setState');

    if (!internalInstance) {
      return;
    }

    // 如果_pendingStateQueue为空,则创建它。可以发现队列是数组形式实现的
    var queue = internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue = []);
    queue.push(partialState);

    // 将要更新的ReactComponent放入数组中
    enqueueUpdate(internalInstance);
}
```
其中getInternalInstanceReadyForUpdate源码如下，解释都在代码注释中

```javascript

function getInternalInstanceReadyForUpdate(publicInstance, callerName) {
  // 从map取出ReactComponent组件,还记得mountComponent时把ReactElement作为key，将ReactComponent存入了map中了吧，ReactComponent是React组件的核心，包含各种状态，数据和操作方法。而ReactElement则仅仅是一个数据类。
  var internalInstance = ReactInstanceMap.get(publicInstance);
  if (!internalInstance) {
    return null;
  }

  return internalInstance;
}
```

enqueueUpdate源码如下
```javascript
function enqueueUpdate(component) {
  ensureInjected();

  // 如果不是正处于创建或更新组件阶段,则处理update事务
  if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }

  // 如果正在创建或更新组件,则暂且先不处理update,只是将组件放在dirtyComponents数组中
  dirtyComponents.push(component);
}

```
enqueueUpdate包含了React避免重复render的逻辑。mountComponent和updateComponent方法在执行的最开始，会调用到batchedUpdates进行批处理更新，此时会将isBatchingUpdates设置为true，也就是将状态标记为现在正处于更新阶段了。之后React以事务的方式处理组件update，事务处理完后会调用wrapper.close(), 而TRANSACTION_WRAPPERS中包含了RESET_BATCHED_UPDATES这个wrapper，故最终会调用RESET_BATCHED_UPDATES.close(), 它最终会将isBatchingUpdates设置为false。这个过程比较麻烦，想更清晰的了解的话，建议自行分析源码。

故getInitialState，componentWillMount， render，componentWillUpdate 中setState都不会引起updateComponent。但在componentDidMount和componentDidUpdate中则会。

```javascript
batchedUpdates: function (callback, a, b, c, d, e) {
  var alreadyBatchingUpdates = ReactDefaultBatchingStrategy.isBatchingUpdates;
  // 批处理最开始时，将isBatchingUpdates设为true，表明正在更新
  ReactDefaultBatchingStrategy.isBatchingUpdates = true;

  // The code is written this way to avoid extra allocations
  if (alreadyBatchingUpdates) {
    callback(a, b, c, d, e);
  } else {
    // 以事务的方式处理updates，后面详细分析transaction
    transaction.perform(callback, null, a, b, c, d, e);
  }
}

var RESET_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: function () {
    // 事务批更新处理结束时，将isBatchingUpdates设为了false
    ReactDefaultBatchingStrategy.isBatchingUpdates = false;
  }
};
var TRANSACTION_WRAPPERS = [FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES];

```

# 事务transaction

事务通过wrapper进行封装。一个wrapper包含一对initialize和close方法。比如RESET_BATCHED_UPDATES

```javascript
var RESET_BATCHED_UPDATES = {
  // 初始化调用
  initialize: emptyFunction,
  // 事务执行完成，close时调用
  close: function () {
    ReactDefaultBatchingStrategy.isBatchingUpdates = false;
  }
};
```

transcation被包装在wrapper中，比如

```javascript
var TRANSACTION_WRAPPERS = [FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES];
```

transaction是通过transaction.perform(callback, args…)方法进入的，它会先调用注册好的wrapper中的initialize方法，然后执行perform方法中的callback，最后再执行close方法。

下面分析transaction.perform(callback, args…)

```javascript
perform: function (method, scope, a, b, c, d, e, f) {
    var errorThrown;
    var ret;
    try {
      this._isInTransaction = true;
      errorThrown = true;
      // 先运行所有wrapper中的initialize方法
      this.initializeAll(0);
      
      // 再执行perform方法传入的callback
      ret = method.call(scope, a, b, c, d, e, f);
      errorThrown = false;
    } finally {
      try {
        if (errorThrown) {
          // 最后运行wrapper中的close方法
          try {
            this.closeAll(0);
          } catch (err) {}
        } else {
          // 最后运行wrapper中的close方法
          this.closeAll(0);
        }
      } finally {
        this._isInTransaction = false;
      }
    }
    return ret;
  },
    
  initializeAll: function (startIndex) {
    var transactionWrappers = this.transactionWrappers;
    // 遍历所有注册的wrapper
    for (var i = startIndex; i < transactionWrappers.length; i++) {
      var wrapper = transactionWrappers[i];
      try {
        this.wrapperInitData[i] = Transaction.OBSERVED_ERROR;
        // 调用wrapper的initialize方法
        this.wrapperInitData[i] = wrapper.initialize ? wrapper.initialize.call(this) : null;
      } finally {
        if (this.wrapperInitData[i] === Transaction.OBSERVED_ERROR) {
          try {
            this.initializeAll(i + 1);
          } catch (err) {}
        }
      }
    }
  },
    
  closeAll: function (startIndex) {
    var transactionWrappers = this.transactionWrappers;
    // 遍历所有wrapper
    for (var i = startIndex; i < transactionWrappers.length; i++) {
      var wrapper = transactionWrappers[i];
      var initData = this.wrapperInitData[i];
      var errorThrown;
      try {
        errorThrown = true;
        if (initData !== Transaction.OBSERVED_ERROR && wrapper.close) {
          // 调用wrapper的close方法，如果有的话
          wrapper.close.call(this, initData);
        }
        errorThrown = false;
      } finally {
        if (errorThrown) {
          try {
            this.closeAll(i + 1);
          } catch (e) {}
        }
      }
    }
    this.wrapperInitData.length = 0;
```

# runBatchedUpdates更新组件
前面分析到enqueueUpdate中调用transaction.perform(callback, args...)后，小伙伴们肯定会发现，callback还是enqueueUpdate方法啊，那岂不是死循环了？不是说好的setState会调用updateComponent，从而自动刷新View的吗？别急，我们还是要先从transaction事务说起。

我们的wrapper中注册了两个wrapper，如下
```javascript
var TRANSACTION_WRAPPERS = [FLUSH_BATCHED_UPDATES, RESET_BATCHED_UPDATES];
```

RESET_BATCHED_UPDATES用来管理isBatchingUpdates状态，我们前面在分析setState是否立即生效时已经讲解过了。那FLUSH_BATCHED_UPDATES用来干嘛呢？

```javascript
var FLUSH_BATCHED_UPDATES = {
  initialize: emptyFunction,
  close: ReactUpdates.flushBatchedUpdates.bind(ReactUpdates)
};

var flushBatchedUpdates = function () {
  // 循环遍历处理完所有dirtyComponents
  while (dirtyComponents.length || asapEnqueued) {
    if (dirtyComponents.length) {
      var transaction = ReactUpdatesFlushTransaction.getPooled();
      // close前执行完runBatchedUpdates方法，这是关键
      transaction.perform(runBatchedUpdates, null, transaction);
      ReactUpdatesFlushTransaction.release(transaction);
    }

    if (asapEnqueued) {
      asapEnqueued = false;
      var queue = asapCallbackQueue;
      asapCallbackQueue = CallbackQueue.getPooled();
      queue.notifyAll();
      CallbackQueue.release(queue);
    }
  }
};
```

FLUSH_BATCHED_UPDATES会在一个transaction的close阶段运行runBatchedUpdates，从而执行update。

```javascript
function runBatchedUpdates(transaction) {
  var len = transaction.dirtyComponentsLength;
  dirtyComponents.sort(mountOrderComparator);

  for (var i = 0; i < len; i++) {
    // dirtyComponents中取出一个component
    var component = dirtyComponents[i];

    // 取出dirtyComponent中的未执行的callback,下面就准备执行它了
    var callbacks = component._pendingCallbacks;
    component._pendingCallbacks = null;

    var markerName;
    if (ReactFeatureFlags.logTopLevelRenders) {
      var namedComponent = component;
      if (component._currentElement.props === component._renderedComponent._currentElement) {
        namedComponent = component._renderedComponent;
      }
    }
    // 执行updateComponent
    ReactReconciler.performUpdateIfNecessary(component, transaction.reconcileTransaction);
    
    // 执行dirtyComponent中之前未执行的callback
    if (callbacks) {
      for (var j = 0; j < callbacks.length; j++) {
        transaction.callbackQueue.enqueue(callbacks[j], component.getPublicInstance());
      }
    }
  }
}
```

runBatchedUpdates循环遍历dirtyComponents数组，主要干两件事。首先执行performUpdateIfNecessary来刷新组件的view，然后执行之前阻塞的callback。下面来看performUpdateIfNecessary。

```javascript
performUpdateIfNecessary: function (transaction) {
    if (this._pendingElement != null) {
      // receiveComponent会最终调用到updateComponent，从而刷新View
      ReactReconciler.receiveComponent(this, this._pendingElement, transaction, this._context);
    }

    if (this._pendingStateQueue !== null || this._pendingForceUpdate) {
      // 执行updateComponent，从而刷新View。这个流程在React生命周期中讲解过
      this.updateComponent(transaction, this._currentElement, this._currentElement, this._context, this._context);
    }
  },
```


最后惊喜的看到了receiveComponent和updateComponent吧。receiveComponent最后会调用updateComponent，而updateComponent中会执行React组件存在期的生命周期方法，如componentWillReceiveProps， shouldComponentUpdate， componentWillUpdate，render, componentDidUpdate。 从而完成组件更新的整套流程。

# 总结
setState流程还是很复杂的，设计也很精巧，避免了重复无谓的刷新组件。它的主要流程如下

> * enqueueSetState将state放入队列中，并调用enqueueUpdate处理要更新的Component
> * 如果组件当前正处于update事务中，则先将Component存入dirtyComponent中。否则调用batchedUpdates处理。
> * batchedUpdates发起一次transaction.perform()事务
> * 开始执行事务初始化，运行，结束三个阶段
1.初始化：事务初始化阶段没有注册方法，故无方法要执行
2.运行：执行setSate时传入的callback方法，一般不会传callback参数
3.结束：更新isBatchingUpdates为false，并执行FLUSH_BATCHED_UPDATES这个wrapper中的close方法
> * FLUSH_BATCHED_UPDATES在close阶段，会循环遍历所有的dirtyComponents，调用updateComponent刷新组件，并执行它的pendingCallbacks, 也就是setState中设置的callback。

[原文地址](https://yq.aliyun.com/articles/72329?spm=a2c4e.11153940.blogcont72330.20.351d5033CZkYc1)