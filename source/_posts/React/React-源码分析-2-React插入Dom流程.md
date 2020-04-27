---
title: React 源码分析 2 React组件插入Dom流程
date: 2018-03-26 10:16:27
tags: "React"
categories: "React"
---

# 简介
React广受好评的一个重要原因就是组件化开发，一方面分模块的方式便于协同开发，降低耦合，后期维护也轻松；另一方面使得一次开发，多处复用成为现实，甚至可以直接复用开源React组件。开发完一个组件后，我们需要插入DOM中，一般使用如下代码

```javascript
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('example')
);
```
经过babel转码后为

```javascript
ReactDOM.render(
        React.createElement(
          'h1',   // type, DOM原生组件的type为string，React自定义组件type为Object
          null,   // config，会设置到ref，key，props中
          'Hello, world!'   // children，子组件.这儿为文本组件
        ),
        document.getElementById('example')
)
```

那么React底层是怎么将组件插入DOM中的呢。本文来详细分析它的前因后果。


# ReactMount._renderSubtreeIntoContainer()
ReactDOM.render()实际调用ReactMount.render(),接着调用到ReactMount._renderSubtreeIntoContainer().

这个调用链比较简单，不分析了。下面重点分析_renderSubtreeIntoContainer(). 我们去除掉开发调试阶段的报错代码（比如 "development" !== 'production'）。

<!-- more -->
```javascript
   /**
   * 将ReactElement插入DOM中，并返回ReactElement对应的ReactComponent。
   * ReactElement是React元素在内存中的表示形式，可以理解为一个数据类，包含type，key，refs，props等成员变量
   * ReactComponent是React元素的操作类，包含mountComponent(), updateComponent()等很多操作组件的方法
   *
   * @param {parentComponent} 父组件，对于第一次渲染，为null
   * @param {nextElement} 要插入到DOM中的组件，对应上面例子中的<h1>Hello, world!</h1>经过babel转译后的元素
   * @param {container} 要插入到的容器，对应上面例子中的document.getElementById('example')获取的DOM对象
   * @param {callback} 第一次渲染为null
   *
   * @return {component}  返回ReactComponent，对于ReactDOM.render()调用，不用管返回值。
   */ 
_renderSubtreeIntoContainer: function (parentComponent, nextElement, container, callback) {
    // 刚开始一段开发阶段的报错代码，省去
    ...

    // 包装ReactElement，将nextElement挂载到wrapper的props属性下，这段代码不是很关键
    var nextWrappedElement = ReactElement(TopLevelWrapper, null, null, null, null, null, nextElement);
    // 获取要插入到的容器的前一次的ReactComponent，这是为了做DOM diff
    // 对于ReactDOM.render()调用，prevComponent为null
    var prevComponent = getTopLevelWrapperInContainer(container);

    if (prevComponent) {
      // 从prevComponent中获取到prevElement这个数据对象。一定要搞清楚ReactElement和ReactComponent的作用，他们很关键
      var prevWrappedElement = prevComponent._currentElement;
      var prevElement = prevWrappedElement.props;
      // DOM diff精髓，同一层级内，type和key不变时，只用update就行。否则先unmount组件再mount组件
      // 这是React为了避免递归太深，而做的DOM diff前提假设。它只对同一DOM层级，type相同，key(如果有)相同的组件做DOM diff，否则不用比较，直接先unmount再mount。这个假设使得diff算法复杂度从O(n^3)降低为O(n).
      // shouldUpdateReactComponent源码请看后面的分析
      if (shouldUpdateReactComponent(prevElement, nextElement)) {
        var publicInst = prevComponent._renderedComponent.getPublicInstance();
        var updatedCallback = callback && function () {
          callback.call(publicInst);
        };
        // 只需要update，调用_updateRootComponent，然后直接return了
        ReactMount._updateRootComponent(prevComponent, nextWrappedElement, container, updatedCallback);
        return publicInst;
      } else {
        // 不做update，直接先卸载再挂载。即unmountComponent,再mountComponent。mountComponent在后面代码中进行
        ReactMount.unmountComponentAtNode(container);
      }
    }

    var reactRootElement = getReactRootElementInContainer(container);
    var containerHasReactMarkup = reactRootElement && !!internalGetID(reactRootElement);
    var containerHasNonRootReactChild = hasNonRootReactChild(container);

    var shouldReuseMarkup = containerHasReactMarkup && !prevComponent && !containerHasNonRootReactChild;
    // 初始化，渲染组件，然后插入到DOM中。_renderNewRootComponent很关键，后面详细分析
    var component = ReactMount._renderNewRootComponent(nextWrappedElement, container, shouldReuseMarkup, parentComponent != null ? parentComponent._reactInternalInstance._processChildContext(parentComponent._reactInternalInstance._context) : emptyObject)._renderedComponent.getPublicInstance();
    // render方法中带入的回调，ReactDOM.render()调用时一般不传入
    if (callback) {
      callback.call(component);
    }
    return component;
  },
```

shouldUpdateReactComponent()源码如下：
```javascript
function shouldUpdateReactComponent(prevElement, nextElement) {
  // 前后两次ReactElement中任何一个为null，则必须另一个为null才返回true。这种情况一般不会碰到
  var prevEmpty = prevElement === null || prevElement === false;
  var nextEmpty = nextElement === null || nextElement === false;
  if (prevEmpty || nextEmpty) {
    return prevEmpty === nextEmpty;
  }

  var prevType = typeof prevElement;
  var nextType = typeof nextElement;

  // React DOM diff算法
  if (prevType === 'string' || prevType === 'number') {
    // 如果前后两次为数字或者字符,则认为只需要update(处理文本元素)，返回true
    return (nextType === 'string' || nextType === 'number');
  } else {
      // 如果前后两次为DOM元素或React元素,则必须type和key不变(key用于listView等组件,很多时候我们没有设置key，故只需type相同)才update,否则先unmount再重新mount。返回false
    return (
      nextType === 'object' &&
      prevElement.type === nextElement.type &&
      prevElement.key === nextElement.key
    );
  }
}
```

# ReactMount._renderNewRootComponent
```javascript
 _renderNewRootComponent: function (nextElement, container, shouldReuseMarkup, context) {
    ReactBrowserEventEmitter.ensureScrollValueMonitoring();
    // 初始化ReactComponent，根据ReactElement中不同的type字段，创建不同类型的组件对象，即ReactComponent
    // 前一篇文章中已经分析了。http://blog.csdn.net/u013510838/article/details/55669769
    var componentInstance = instantiateReactComponent(nextElement);

    // 处理batchedMountComponentIntoNode方法调用，将ReactComponent插入DOM中，后面详细分析
    ReactUpdates.batchedUpdates(batchedMountComponentIntoNode, componentInstance, container, shouldReuseMarkup, context);

    var wrapperID = componentInstance._instance.rootID;
    instancesByReactRootID[wrapperID] = componentInstance;

    return componentInstance;
  },
```

batchedMountComponentIntoNode以transaction事务的形式调用mountComponentIntoNode，源码分析如下。

```javascript
function mountComponentIntoNode(wrapperInstance, container, transaction, shouldReuseMarkup, context) {
  var markerName;
  // 一段log，可以不管
  if (ReactFeatureFlags.logTopLevelRenders) {
    var wrappedElement = wrapperInstance._currentElement.props;
    var type = wrappedElement.type;
    markerName = 'React mount: ' + (typeof type === 'string' ? type : type.displayName || type.name);
    console.time(markerName);
  }

  // 调用对应ReactComponent中的mountComponent方法来渲染组件，这个是React生命周期的重要方法。后面详细分析。
  // mountComponent返回React组件解析的HTML。不同的ReactComponent的mountComponent策略不同，可以看做多态
  // 上面的<h1>Hello, world!</h1>, 对应的是ReactDOMTextComponent，最终解析成的HTML为
  // <h1 data-reactroot="">Hello, world!</h1>
  var markup = ReactReconciler.mountComponent(wrapperInstance, transaction, null, ReactDOMContainerInfo(wrapperInstance, container), context);

  if (markerName) {
    console.timeEnd(markerName);
  }

  wrapperInstance._renderedComponent._topLevelWrapper = wrapperInstance;
  // 将解析出来的HTML插入DOM中
  ReactMount._mountImageIntoNode(markup, container, wrapperInstance, shouldReuseMarkup, transaction);
}
```

_mountImageIntoNode源码如下

```javascript
_mountImageIntoNode: function (markup, container, instance, shouldReuseMarkup, transaction) {
    // 对于ReactDOM.render()调用，shouldReuseMarkup为false
    if (shouldReuseMarkup) {
      ...
    }

    if (transaction.useCreateElement) {
      // 清空container的子节点，这个地方不明白为什么这么做
      while (container.lastChild) {
        container.removeChild(container.lastChild);
      }
      DOMLazyTree.insertTreeBefore(container, markup, null);
    } else {
      // 将markup这个HTML设置到container这个DOM元素的innerHTML属性上，这样就插入到了DOM中了
      setInnerHTML(container, markup);
      // 将instance这个ReactComponent渲染后的对象，即Virtual DOM，保存到container这个DOM元素的firstChild这个原生节点上。简单理解就是将Virtual DOM保存到内存中，这样可以大大提高交互效率
      ReactDOMComponentTree.precacheNode(instance, container.firstChild);
    }
  }
```

# 总结
ReactDOM.render(）是渲染React组件并插入到DOM中的入口方法，它的执行流程大概为

1.React.createElement(),创建ReactElement对象。他的重要的成员变量有type,key,ref,props。这个过程中会调用getInitialState(), 初始化state，只在挂载的时候才调用。后面update时不再调用了。

2.instantiateReactComponent()，根据ReactElement的type分别创建ReactDOMComponent， ReactCompositeComponent，ReactDOMTextComponent等对象

3.mountComponent(), 调用React生命周期方法解析组件，得到它的HTML。

4._mountImageIntoNode(), 将HTML插入到DOM父节点中，通过设置DOM父节点的innerHTML属性。

5.缓存节点在React中的对应对象，即Virtual DOM。

!()[]

[原文地址](https://zhuanlan.zhihu.com/p/25882202)