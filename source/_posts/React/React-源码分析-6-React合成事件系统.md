---
title: React 源码分析 6 React合成事件系统
date: 2018-03-27 15:36:12
tags: "React"
categories: "React"
---

# React合成事件特点
React自己实现了一套高效的事件注册，存储，分发和重用逻辑，在DOM事件体系基础上做了很大改进，减少了内存消耗，简化了事件逻辑，并最大化的解决了IE等浏览器的不兼容问题。与DOM事件体系相比，它有如下特点
> * 1.React组件上声明的事件最终绑定到了document这个DOM节点上，而不是React组件对应的DOM节点。故只有document这个节点上面才绑定了DOM原生事件，其他节点没有绑定事件。这样简化了DOM原生事件，减少了内存开销

> * 2.React以队列的方式，从触发事件的组件向父组件回溯，调用它们在JSX中声明的callback。也就是React自身实现了一套事件冒泡机制。我们没办法用event.stopPropagation()来停止事件传播，应该使用event.preventDefault()

> * 3.React有一套自己的合成事件SyntheticEvent，不同类型的事件会构造不同的SyntheticEvent

> * 4.React使用对象池来管理合成事件对象的创建和销毁，这样减少了垃圾的生成和新对象内存的分配，大大提高了性能


那么这些特性是如何实现的呢，下面和大家一起一探究竟。

# React事件系统
先看Facebook给出的React事件系统框图
![reactimg](https://pic4.zhimg.com/80/v2-e1dc82ada4fbbf5c366558532a6f6fca_hd.jpg)

浏览器事件（如用户点击了某个button）触发后，DOM将event传给ReactEventListener，它将事件分发到当前组件及以上的父组件。然后由ReactEventEmitter对每个组件进行事件的执行，先构造React合成事件，然后以queue的方式调用JSX中声明的callback进行事件回调。

涉及到的主要类如下

**ReactEventListene**:负责事件注册和事件分发。React将DOM事件全都注册到document这个节点上，这个我们在事件注册小节详细讲。事件分发主要调用dispatchEvent进行，从事件触发组件开始，向父元素遍历。我们在事件执行小节详细讲。

**ReactEventEmitter**：负责每个组件上事件的执行。

**EventPluginHub**：负责事件的存储，合成事件以对象池的方式实现创建和销毁，大大提高了性能。

**SimpleEventPlugin等plugin**：根据不同的事件类型，构造不同的合成事件。如focus对应的React合成事件为SyntheticFocusEvent

<!-- more -->

# 事件注册
JSX中声明一个React事件十分简单，比如
```javascript
render() {
  return (
    <div onClick = {
            (event) => {console.log(JSON.stringify(event))}
        } 
    />
  );
}
```

那么它是如何被注册到React事件系统中的呢？


还是先得从组件创建和更新的入口方法mountComponent和updateComponent说起。在这两个方法中，都会调用到_updateDOMProperties方法，对JSX中声明的组件属性进行处理。源码如下

```javascript
_updateDOMProperties: function (lastProps, nextProps, transaction) {
    ... // 前面代码太长，省略一部分
    else if (registrationNameModules.hasOwnProperty(propKey)) {
        // 如果是props这个对象直接声明的属性，而不是从原型链中继承而来的，则处理它
        // nextProp表示要创建或者更新的属性，而lastProp则表示上一次的属性
        // 对于mountComponent，lastProp为null。updateComponent二者都不为null。unmountComponent则nextProp为null
        if (nextProp) {
          // mountComponent和updateComponent中，enqueuePutListener注册事件
          enqueuePutListener(this, propKey, nextProp, transaction);
        } else if (lastProp) {
          // unmountComponent中，删除注册的listener，防止内存泄漏
          deleteListener(this, propKey);
        }
    }
}
```

下面我们来看enqueuePutListener，它负责注册JSX中声明的事件。源码如下

```javascript
// inst: React Component对象
// registrationName: React合成事件名，如onClick
// listener: React事件回调方法，如onClick=callback中的callback
// transaction: mountComponent或updateComponent所处的事务流中，React都是基于事务流的
function enqueuePutListener(inst, registrationName, listener, transaction) {
  if (transaction instanceof ReactServerRenderingTransaction) {
    return;
  }
  var containerInfo = inst._hostContainerInfo;
  var isDocumentFragment = containerInfo._node && containerInfo._node.nodeType === DOC_FRAGMENT_TYPE;
  // 找到document
  var doc = isDocumentFragment ? containerInfo._node : containerInfo._ownerDocument;
  // 注册事件，将事件注册到document上
  listenTo(registrationName, doc);
  // 存储事件,放入事务队列中
  transaction.getReactMountReady().enqueue(putListener, {
    inst: inst,
    registrationName: registrationName,
    listener: listener
  });
}
```

enqueuePutListener主要做两件事，一方面将事件注册到document这个原生DOM上（这就是为什么只有document这个节点有DOM事件的原因），另一方面采用事务队列的方式调用putListener将注册的事件存储起来，以供事件触发时回调。

注册事件的入口是listenTo方法, 它解决了不同浏览器间捕获和冒泡不兼容的问题。事件回调方法在bubble阶段被触发。如果我们想让它在capture阶段触发，则需要在事件名上加上capture。比如onClick在bubble阶段触发，而onCaptureClick在capture阶段触发。listenTo代码虽然比较长，但逻辑很简单，调用trapCapturedEvent和trapBubbledEvent来注册捕获和冒泡事件。trapCapturedEvent大家可以自行分析，我们仅分析trapBubbledEvent，如下

```javascript
trapBubbledEvent: function (topLevelType, handlerBaseName, element) {
    if (!element) {
      return null;
    }
    return EventListener.listen(
      element,   // 绑定到的DOM目标,也就是document
      handlerBaseName,   // eventType
      ReactEventListener.dispatchEvent.bind(null, topLevelType));  // callback, document上的原生事件触发后回调
  },
    
  listen: function listen(target, eventType, callback) {
    if (target.addEventListener) {
      // 将原生事件添加到target这个dom上,也就是document上。
      // 这就是只有document这个DOM节点上有原生事件的原因
      target.addEventListener(eventType, callback, false);
      return {
        // 删除事件,这个由React自己回调,不需要调用者来销毁。但仅仅对于React合成事件才行
        remove: function remove() {
          target.removeEventListener(eventType, callback, false);
        }
      };
    } else if (target.attachEvent) {
      // attach和detach的方式
      target.attachEvent('on' + eventType, callback);
      return {
        remove: function remove() {
          target.detachEvent('on' + eventType, callback);
        }
      };
    }
  },
```

在listen方法中，我们终于发现了熟悉的addEventListener这个原生事件注册方法。只有document节点才会调用这个方法，故仅仅只有document节点上才有DOM事件。这大大简化了DOM事件逻辑，也节约了内存。

# 事件存储
事件存储由EventPluginHub来负责，它的入口在我们上面讲到的enqueuePutListener中的putListener方法，如下

```javascript
  /**
   * EventPluginHub用来存储React事件, 将listener存储到`listenerBank[registrationName][key]`
   *
   * @param {object} inst: 事件源
   * @param {string} listener的名字,比如onClick
   * @param {function} listener的callback
   */
  //
  putListener: function (inst, registrationName, listener) {

    // 用来标识注册了事件,比如onClick的React对象。key的格式为'.nodeId', 只用知道它可以标示哪个React对象就可以了
    var key = getDictionaryKey(inst);
    var bankForRegistrationName = listenerBank[registrationName] || (listenerBank[registrationName] = {});
    // 将listener事件回调方法存入listenerBank[registrationName][key]中,比如listenerBank['onclick'][nodeId]
    // 所有React组件对象定义的所有React事件都会存储在listenerBank中
    bankForRegistrationName[key] = listener;

    //onSelect和onClick注册了两个事件回调插件, 用于walkAround某些浏览器兼容bug,不用care
    var PluginModule = EventPluginRegistry.registrationNameModules[registrationName];
    if (PluginModule && PluginModule.didPutListener) {
      PluginModule.didPutListener(inst, registrationName, listener);
    }
  },
    
var getDictionaryKey = function (inst) {
  return '.' + inst._rootNodeID;
};
```

由上可见，事件存储在了listenerBank对象中，它按照事件名和React组件对象进行了二维划分，比如nodeId组件上注册的onClick事件最后存储在listenerBank.onclick[nodeId]中

# 事件执行

## 事件分发
当事件触发时，document上addEventListener注册的callback会被回调。从前面事件注册部分发现，此时回调函数为ReactEventListener.dispatchEvent，它是事件分发的入口方法。下面我们来详细分析

![react3](https://pic2.zhimg.com/80/v2-35d1c1174231dd45f3fda05516ed0239_hd.jpg)

```javascript
// topLevelType：带top的事件名，如topClick。不用纠结为什么带一个top字段，知道它是事件名就OK了
// nativeEvent: 用户触发click等事件时，浏览器传递的原生事件
dispatchEvent: function (topLevelType, nativeEvent) {
    // disable了则直接不回调相关方法
    if (!ReactEventListener._enabled) {
      return;
    }

    var bookKeeping = TopLevelCallbackBookKeeping.getPooled(topLevelType, nativeEvent);
    try {
      // 放入批处理队列中,React事件流也是一个消息队列的方式
      ReactUpdates.batchedUpdates(handleTopLevelImpl, bookKeeping);
    } finally {
      TopLevelCallbackBookKeeping.release(bookKeeping);
    }
}
```


可见我们仍然使用批处理的方式进行事件分发，handleTopLevelImpl才是事件分发的真正执行者，它是事件分发的核心，体现了React事件分发的特点，如下

```javascript
// document进行事件分发,这样具体的React组件才能得到响应。因为DOM事件是绑定到document上的
function handleTopLevelImpl(bookKeeping) {
  // 找到事件触发的DOM和React Component
  var nativeEventTarget = getEventTarget(bookKeeping.nativeEvent);
  var targetInst = ReactDOMComponentTree.getClosestInstanceFromNode(nativeEventTarget);

  // 执行事件回调前,先由当前组件向上遍历它的所有父组件。得到ancestors这个数组。
  // 因为事件回调中可能会改变Virtual DOM结构,所以要先遍历好组件层级
  var ancestor = targetInst;
  do {
    bookKeeping.ancestors.push(ancestor);
    ancestor = ancestor && findParent(ancestor);
  } while (ancestor);

  // 从当前组件向父组件遍历,依次执行注册的回调方法. 我们遍历构造ancestors数组时,是从当前组件向父组件回溯的,故此处事件回调也是这个顺序
  // 这个顺序就是冒泡的顺序,并且我们发现不能通过stopPropagation来阻止'冒泡'。
  for (var i = 0; i < bookKeeping.ancestors.length; i++) {
    targetInst = bookKeeping.ancestors[i];
    ReactEventListener._handleTopLevel(bookKeeping.topLevelType, targetInst, bookKeeping.nativeEvent, getEventTarget(bookKeeping.nativeEvent));
  }
}
```


从上面的事件分发中可见，React自身实现了一套冒泡机制。从触发事件的对象开始，向父元素回溯，依次调用它们注册的事件callback

## 事件callback 调用
事件处理由_handleTopLevel完成。它其实是调用ReactBrowserEventEmitter.handleTopLevel() ，如下

```javascript
// React事件调用的入口。DOM事件绑定在了document原生对象上,每次事件触发,都会调用到handleTopLevel
  handleTopLevel: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
    // 采用对象池的方式构造出合成事件。不同的eventType的合成事件可能不同
    var events = EventPluginHub.extractEvents(topLevelType, targetInst, nativeEvent, nativeEventTarget);
    // 批处理队列中的events
    runEventQueueInBatch(events);
  }
```

handleTopLevel方法是事件callback调用的核心。它主要做两件事情，一方面利用浏览器回传的原生事件构造出React合成事件，另一方面采用队列的方式处理events。先看如何构造合成事件。

### 构造合成事件
```javascript
 // 构造合成事件
  extractEvents: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
    var events;
    // EventPluginHub可以存储React合成事件的callback,也存储了一些plugin,这些plugin在EventPluginHub初始化时就注册就来了
    var plugins = EventPluginRegistry.plugins;
    for (var i = 0; i < plugins.length; i++) {
      var possiblePlugin = plugins[i];
      if (possiblePlugin) {
        // 根据eventType构造不同的合成事件SyntheticEvent
        var extractedEvents = possiblePlugin.extractEvents(topLevelType, targetInst, nativeEvent, nativeEventTarget);
        if (extractedEvents) {
          // 将构造好的合成事件extractedEvents添加到events数组中,这样就保存了所有plugin构造的合成事件
          events = accumulateInto(events, extractedEvents);
        }
      }
    }
    return events;
  },
```

EventPluginRegistry.plugins默认包含五种plugin，他们是在EventPluginHub初始化阶段注入进去的，且看代码

```javascript
  // 将eventPlugin注册到EventPluginHub中
  ReactInjection.EventPluginHub.injectEventPluginsByName({
    SimpleEventPlugin: SimpleEventPlugin,
    EnterLeaveEventPlugin: EnterLeaveEventPlugin,
    ChangeEventPlugin: ChangeEventPlugin,
    SelectEventPlugin: SelectEventPlugin,
    BeforeInputEventPlugin: BeforeInputEventPlugin
  });
```

不同的plugin针对不同的事件有特殊的处理，此处我们不展开讲了，下面仅分析SimpleEventPlugin中方法即可。

我们先看SimpleEventPlugin如何构造它所对应的React合成事件。

```javascript
  // 根据不同事件类型,比如click,focus构造不同的合成事件SyntheticEvent, 如SyntheticKeyboardEvent SyntheticFocusEvent
extractEvents: function (topLevelType, targetInst, nativeEvent, nativeEventTarget) {
    var dispatchConfig = topLevelEventsToDispatchConfig[topLevelType];
    if (!dispatchConfig) {
      return null;
    }
    var EventConstructor;
  
   // 根据事件类型，采用不同的SyntheticEvent来构造不同的合成事件
    switch (topLevelType) {
      ... // 省略一些事件，我们仅以blur和focus为例
      case 'topBlur':
      case 'topFocus':
        EventConstructor = SyntheticFocusEvent;
        break;
      ... // 省略一些事件
    }

    // 从event对象池中取出合成事件对象,利用对象池思想,可以大大降低对象创建和销毁的时间,提高性能。这是React事件系统的一大亮点
    var event = EventConstructor.getPooled(dispatchConfig, targetInst, nativeEvent, nativeEventTarget);
    EventPropagators.accumulateTwoPhaseDispatches(event);
    return event;
},
```
这里我们看到了event对象池这个重大特性，采用合成事件对象池的方式，可以大大降低销毁和创建合成事件带来的性能开销。

对象创建好之后，我们还会将它添加到events这个队列中，因为事件回调的时候会用到这个队列。添加到events中使用的是accumulateInto方法。它思路比较简单，将新创建的合成对象的引用添加到之前创建好的events队列中即可，源码如下

```javascript
function accumulateInto(current, next) {

  if (current == null) {
    return next;
  }

  // 将next添加到current中,返回一个包含他们两个的新数组
  // 如果next是数组,current不是数组,采用push方法,否则采用concat方法
  // 如果next不是数组,则返回一个current和next构成的新数组
  if (Array.isArray(current)) {
    if (Array.isArray(next)) {
      current.push.apply(current, next);
      return current;
    }
    current.push(next);
    return current;
  }

  if (Array.isArray(next)) {
    return [current].concat(next);
  }

  return [current, next];
}
```

### 批量处理合成事件
我们上面分析过了，React以队列的形式处理合成事件。方法入口为runEventQueueInBatch，如下

```javascript

  function runEventQueueInBatch(events) {
      // 先将events事件放入队列中
      EventPluginHub.enqueueEvents(events);
      // 再处理队列中的事件,包括之前未处理完的。先入先处理原则
      EventPluginHub.processEventQueue(false);
  }

  /**
   * syntheticEvent放入队列中,等到processEventQueue再获得执行
   */
  enqueueEvents: function (events) {
    if (events) {
      eventQueue = accumulateInto(eventQueue, events);
    }
  },
    
  /**
   * 分发执行队列中的React合成事件。React事件是采用消息队列方式批处理的
   *
   * simulated：为true表示React测试代码，我们一般都是false 
   */
  processEventQueue: function (simulated) {
    // 先将eventQueue重置为空
    var processingEventQueue = eventQueue;
    eventQueue = null;
    if (simulated) {
      forEachAccumulated(processingEventQueue, executeDispatchesAndReleaseSimulated);
    } else {
      // 遍历处理队列中的事件,
      // 如果只有一个元素,则直接executeDispatchesAndReleaseTopLevel(processingEventQueue)
      // 否则遍历队列中事件,调用executeDispatchesAndReleaseTopLevel处理每个元素
      forEachAccumulated(processingEventQueue, executeDispatchesAndReleaseTopLevel);
    }
    // This would be a good time to rethrow if any of the event handlers threw.
    ReactErrorUtils.rethrowCaughtError();
  },

```

合成事件处理也分为两步，先将我们要处理的events队列放入eventQueue中，因为之前可能就存在还没处理完的合成事件。然后再执行eventQueue中的事件。可见，如果之前有事件未处理完，这里就又有得到执行的机会了。

事件执行的入口方法为executeDispatchesAndReleaseTopLevel，如下

```javascript
var executeDispatchesAndReleaseTopLevel = function (e) {
  return executeDispatchesAndRelease(e, false);
};

var executeDispatchesAndRelease = function (event, simulated) {
  if (event) {
    // 进行事件分发,
    EventPluginUtils.executeDispatchesInOrder(event, simulated);

    if (!event.isPersistent()) {
      // 处理完,则release掉event对象,采用对象池方式,减少GC
      // React帮我们处理了合成事件的回收机制，不需要我们关心。但要注意，如果使用了DOM原生事件，则要自己回收
      event.constructor.release(event);
    }
  }
};

// 事件处理的核心
function executeDispatchesInOrder(event, simulated) {
  var dispatchListeners = event._dispatchListeners;
  var dispatchInstances = event._dispatchInstances;

  if (Array.isArray(dispatchListeners)) {
    // 如果有多个listener,则遍历执行数组中event
    for (var i = 0; i < dispatchListeners.length; i++) {
      // 如果isPropagationStopped设成true了,则停止事件传播,退出循环。
      if (event.isPropagationStopped()) {
        break;
      }
      // 执行event的分发,从当前触发事件元素向父元素遍历
      // event为浏览器上传的原生事件
      // dispatchListeners[i]为JSX中声明的事件callback
      // dispatchInstances[i]为对应的React Component 
      executeDispatch(event, simulated, dispatchListeners[i], dispatchInstances[i]);
    }
  } else if (dispatchListeners) {
    // 如果只有一个listener,则直接执行事件分发
    executeDispatch(event, simulated, dispatchListeners, dispatchInstances);
  }
  // 处理完event,重置变量。因为使用的对象池,故必须重置,这样才能被别人复用
  event._dispatchListeners = null;
  event._dispatchInstances = null;
}

```


executeDispatchesInOrder会先得到event对应的listeners队列，然后从当前元素向父元素遍历执行注册的callback。且看executeDispatch

```javascript
function executeDispatch(event, simulated, listener, inst) {
  var type = event.type || 'unknown-event';
  event.currentTarget = EventPluginUtils.getNodeFromInstance(inst);
  if (simulated) {
    // test代码使用,支持try-catch,其他就没啥区别了
    ReactErrorUtils.invokeGuardedCallbackWithCatch(type, listener, event);
  } else {
    // 事件分发,listener为callback,event为参数,类似listener(event)这个方法调用
    // 这样就回调到了我们在JSX中注册的callback。比如onClick={(event) => {console.log(1)}}
    // 这样应该就明白了callback怎么被调用的,以及event参数怎么传入callback里面的了
    ReactErrorUtils.invokeGuardedCallback(type, listener, event);
  }
  event.currentTarget = null;
}

// 采用func(a)的方式进行调用，
// 故ReactErrorUtils.invokeGuardedCallback(type, listener, event)最终调用的是listener(event)
// event对象为浏览器传递的DOM原生事件对象，这也就解释了为什么React合成事件回调中能拿到原生event的原因
function invokeGuardedCallback(name, func, a) {
  try {
    func(a);
  } catch (x) {
    if (caughtError === null) {
      caughtError = x;
    }
  }
}
```

# 总结

React事件系统还是相当麻烦的，主要分为事件注册，事件存储和事件执行三大部分。了解了React事件系统源码，就能够轻松回答我们文章开头所列出的React事件几大特点了。

[原文地址](https://yq.aliyun.com/articles/72421?spm=a2c4e.11153940.blogcont72750.23.2375fe1fR29Lpk)
