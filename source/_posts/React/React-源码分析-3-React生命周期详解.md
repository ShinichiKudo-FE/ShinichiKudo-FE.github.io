---
title: React 源码分析 3 React生命周期详解
date: 2018-03-26 10:18:22
tags: "React"
categories: "React"
---

# React生命周期流程
![react](http://img.colabug.com/2017/10/965595492cafc7b07b8340cdc9b4441d.png)
调用流程可以参看上图。分为实例化，存在期和销毁三个不同阶段。介绍生命周期流程的文章很多，相信大部分同学也有所了解，我们就不详细分析了。很多同学肯定有疑问，这些方法在React内部是在哪些方法中被调用的呢，他们触发的时机又是什么时候呢。下面我们来详细分析。

# 实例化生命周期

## getDefaultProps
在React.creatClass()初始化组件类时，会调用getDefaultProps()，将返回的默认属性挂载到defaultProps变量下

这里要提的一点是，初始化组件类只运行一次。可以把它简单类比为Java中的Class对象。初始化组件类就是ClassLoader加载Class对象的过程。类对象的初始化不要错误理解成了实例对象的初始化。一个React组件类可能会在JSX中被多次调用，产生多个组件对象，但它只有一个类对象，也就是类加载后getDefaultProps就不会再调用了。

## getlnitialState
这个方法在创建组件实例对象的时候被调用，具体代码位于React.creatClass()的Constructor函数中。

每次创建React实例对象时，它都会被调用。

## mountComponent

componentWillMount，render，componentDidMount都是在mountComponent中被调用.它是在渲染新的ReactComponent中被调用的。输入ReactComponent，返回组件对应的HTML。把这个HTML插入到DOM中，就可以生成组件对应的DOM对象了。所以mountComponent尤其关键。

和Java中的多态一样，不同的React组件的mountComponent实现都有所区别。下面我们来重点分析React自定义组件类，也就是ReactCompositeComponent的mountComponent。

```javascript
 mountComponent: function (transaction, nativeParent, nativeContainerInfo, context) {
    this._context = context;
    this._mountOrder = nextMountID++;
    this._nativeParent = nativeParent;
    this._nativeContainerInfo = nativeContainerInfo;

    // 做propTypes是否合法的判断，这个只在开发阶段有用
    var publicProps = this._processProps(this._currentElement.props);
    var publicContext = this._processContext(context);

    var Component = this._currentElement.type;

    // 初始化公共类
    var inst = this._constructComponent(publicProps, publicContext);
    var renderedElement;

    // inst或者inst.render为空对应的是stateless组件，也就是无状态组件
    // 无状态组件没有实例对象，它本质上只是一个返回JSX的函数而已。是一种轻量级的React组件
    if (!shouldConstruct(Component) && (inst == null || inst.render == null)) {
      renderedElement = inst;
      warnIfInvalidElement(Component, renderedElement);
      inst = new StatelessComponent(Component);
    }

    // 设置变量
    inst.props = publicProps;
    inst.context = publicContext;
    inst.refs = emptyObject;
    inst.updater = ReactUpdateQueue;
    this._instance = inst;

    // 存储实例对象的引用到map中，方便以后查找
    ReactInstanceMap.set(inst, this);
    
    // 初始化state，队列等
    var initialState = inst.state;
    if (initialState === undefined) {
      inst.state = initialState = null;
    }
    this._pendingStateQueue = null;
    this._pendingReplaceState = false;
    this._pendingForceUpdate = false;

    var markup;
    if (inst.unstable_handleError) {
      // 挂载时出错，进行一些错误处理，然后performInitialMount，初始化挂载
      markup = this.performInitialMountWithErrorHandling(renderedElement, nativeParent, nativeContainerInfo, transaction, context);
    } else {
      // 初始化挂载
      markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction, context);
    }

    if (inst.componentDidMount) {
      // 调用componentDidMount，以事务的形式
      transaction.getReactMountReady().enqueue(inst.componentDidMount, inst);
    }

    return markup;
  },
```
<!-- more -->
mountComponent先做实例对象的props,state等初始化，然后调用performInitialMount初始化挂载，完成后调用componentDidMount。这个调用链还是很清晰的。下面我们重点来分析performInitialMountWithErrorHandling和performInitialMount

```javascript
 performInitialMountWithErrorHandling: function (renderedElement, nativeParent, nativeContainerInfo, transaction, context) {
    var markup;
    var checkpoint = transaction.checkpoint();
    try {
      // 放到try-catch中，如果没有出错则调用performInitialMount初始化挂载。可见这里没有什么特别的操作，也就是做一些错误处理而已
      markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction, context);
    } catch (e) {
      // handleError，卸载组件，然后重新performInitialMount初始化挂载
      transaction.rollback(checkpoint);
      this._instance.unstable_handleError(e);
      if (this._pendingStateQueue) {
        this._instance.state = this._processPendingState(this._instance.props, this._instance.context);
      }
      checkpoint = transaction.checkpoint();

      this._renderedComponent.unmountComponent(true);
      transaction.rollback(checkpoint);
      
      markup = this.performInitialMount(renderedElement, nativeParent, nativeContainerInfo, transaction, context);
    }
    return markup;
  },
```

可见performInitialMountWithErrorHandling只是多了一层错误处理而已，关键还是在performInitialMount中。
```javascript
performInitialMount: function (renderedElement, nativeParent, nativeContainerInfo, transaction, context) {
    var inst = this._instance;
    if (inst.componentWillMount) {
      // render前调用componentWillMount
      inst.componentWillMount();
      // 将state提前合并，故在componentWillMount中调用setState不会触发重新render，而是做一次state合并。这样做的目的是减少不必要的重新渲染
      if (this._pendingStateQueue) {
        inst.state = this._processPendingState(inst.props, inst.context);
      }
    }

    // 如果不是stateless，即无状态组件，则调用render，返回ReactElement
    if (renderedElement === undefined) {
      renderedElement = this._renderValidatedComponent();
    }

    // 得到组件类型，如空组件ReactNodeTypes.EMPTY，自定义React组件ReactNodeTypes.COMPOSITE，DOM原生组件ReactNodeTypes.NATIVE
    this._renderedNodeType = ReactNodeTypes.getType(renderedElement);
    // 由ReactElement生成ReactComponent，这个方法在之前讲解过。根据不同type创建不同Component对象
    // 参考 http://blog.csdn.net/u013510838/article/details/55669769
    this._renderedComponent = this._instantiateReactComponent(renderedElement);

    // 递归渲染，渲染子组件
    var markup = ReactReconciler.mountComponent(this._renderedComponent, transaction, nativeParent, nativeContainerInfo, this._processChildContext(context));

    return markup;
  },
```


performInitialMount中先调用componentWillMount()，再将setState()产生的state改变进行state合并，然后调用_renderValidatedComponent()返回ReactElement，它会调用render()方法。然后由ReactElement创建ReactComponent。最后进行递归渲染。下面来看renderValidatedComponent()

```javascript
  _renderValidatedComponent: function () {
    var renderedComponent;
    ReactCurrentOwner.current = this;
    try {
      renderedComponent = this._renderValidatedComponentWithoutOwnerOrContext();
    } finally {
      ReactCurrentOwner.current = null;
    }
    !(
    return renderedComponent;
  },
      
  _renderValidatedComponentWithoutOwnerOrContext: function () {
    var inst = this._instance;
    // 调用render方法，得到ReactElement。JSX经过babel转译后其实就是createElement()方法。这一点在前面也讲解过
    var renderedComponent = inst.render();
    return renderedComponent;
  },
```

# 存在期生命周期

组件实例对象已经生成时，我们可以通过setState()来更新组件。setState机制后面会有单独文章分析，现在只用知道它会调用updateComponent()来完成更新即可。下面来分析updateComponent

```javascript
updateComponent: function(transaction, prevParentElement, nextParentElement, prevUnmaskedContext, nextUnmaskedContext
  ) {
    var inst = this._instance;
    var willReceive = false;
    var nextContext;
    var nextProps;

    // context对象如果有改动,则检查propTypes等,这在开发阶段可以报错提醒
    if (this._context === nextUnmaskedContext) {
      nextContext = inst.context;
    } else {
      nextContext = this._processContext(nextUnmaskedContext);
      willReceive = true;
    }

    // 如果父元素类型相同,则跳过propTypes类型检查
    if (prevParentElement === nextParentElement) {
      nextProps = nextParentElement.props;
    } else {
      nextProps = this._processProps(nextParentElement.props);
      willReceive = true;
    }

    // 调用componentWillReceiveProps,如果通过setState进入的updateComponent，则没有这一步
    if (willReceive && inst.componentWillReceiveProps) {
      inst.componentWillReceiveProps(nextProps, nextContext);
    }

    // 提前合并state,componentWillReceiveProps中调用setState不会重新渲染,在此处做合并即可,因为后面也是要调用render的
    // 这样可以避免没必要的渲染
    var nextState = this._processPendingState(nextProps, nextContext);
    
    // 调用shouldComponentUpdate给shouldUpdate赋值
    // 如果通过forceUpdate进入的updateComponent，即_pendingForceUpdate不为空，则不用判断shouldComponentUpdate.
    var shouldUpdate = true;
    if (!this._pendingForceUpdate && inst.shouldComponentUpdate) {
      shouldUpdate = inst.shouldComponentUpdate(nextProps, nextState, nextContext);
    }
    
    // 如果shouldUpdate为true,则会执行渲染,否则不会
    this._updateBatchNumber = null;
    if (shouldUpdate) {
      this._pendingForceUpdate = false;
      // 执行更新渲染,后面详细分析
      this._performComponentUpdate(
        nextParentElement,
        nextProps,
        nextState,
        nextContext,
        transaction,
        nextUnmaskedContext
      );
    } else {
      // shouldUpdate为false,则不会更新渲染
      this._currentElement = nextParentElement;
      this._context = nextUnmaskedContext;
      inst.props = nextProps;
      inst.state = nextState;
      inst.context = nextContext;
    }
},
```


updateComponent中，先调用componentWillReceiveProps，然后合并setState导致的state变化。然后调用shouldComponentUpdate判断是否需要更新渲染。如果需要，则调用_performComponentUpdate执行渲染更新，下面接着分析performComponentUpdate。

```javascript
_performComponentUpdate: function(nextElement,nextProps,nextState,nextContext,transaction,
                                     unmaskedContext
  ) {
    var inst = this._instance;

    // 判断是否已经update了
    var hasComponentDidUpdate = Boolean(inst.componentDidUpdate);
    var prevProps;
    var prevState;
    var prevContext;
    if (hasComponentDidUpdate) {
      prevProps = inst.props;
      prevState = inst.state;
      prevContext = inst.context;
    }

    // render前调用componentWillUpdate
    if (inst.componentWillUpdate) {
      inst.componentWillUpdate(nextProps, nextState, nextContext);
    }

    // state props等属性设置到内部变量inst上
    this._currentElement = nextElement;
    this._context = unmaskedContext;
    inst.props = nextProps;
    inst.state = nextState;
    inst.context = nextContext;

    // 内部会调用render方法，重新解析ReactElement并得到HTML
    this._updateRenderedComponent(transaction, unmaskedContext);

    // render后调用componentDidUpdate
    if (hasComponentDidUpdate) {
      transaction.getReactMountReady().enqueue(
        inst.componentDidUpdate.bind(inst, prevProps, prevState, prevContext),
        inst
      );
    }
},
```

_performComponentUpdate会调用componentWillUpdate，然后在调用updateRenderedComponent进行更新渲染，最后调用componentDidUpdate。下面来看看updateRenderedComponent中怎么调用render方法的。

```javascript
_updateRenderedComponent: function(transaction, context) {
    var prevComponentInstance = this._renderedComponent;
    var prevRenderedElement = prevComponentInstance._currentElement;
    
    // _renderValidatedComponent内部会调用render,得到ReactElement
    var nextRenderedElement = this._renderValidatedComponent();
    
    // 判断是否做DOM diff。React为了简化递归diff,认为组件层级不变,且type和key不变(key用于listView等组件,很多时候我们没有设置type)才update,否则先unmount再重新mount
    if (shouldUpdateReactComponent(prevRenderedElement, nextRenderedElement)) {
      // 递归updateComponent,更新子组件的Virtual DOM
      ReactReconciler.receiveComponent(prevComponentInstance, nextRenderedElement, transaction, this._processChildContext(context));
    } else {
      var oldNativeNode = ReactReconciler.getNativeNode(prevComponentInstance);
      
      // 不做DOM diff,则先卸载掉,然后再加载。也就是先unMountComponent,再mountComponent
      ReactReconciler.unmountComponent(prevComponentInstance, false);

      this._renderedNodeType = ReactNodeTypes.getType(nextRenderedElement);
      
      // 由ReactElement创建ReactComponent
      this._renderedComponent = this._instantiateReactComponent(nextRenderedElement);

      // mountComponent挂载组件,得到组件对应HTML
      var nextMarkup = ReactReconciler.mountComponent(this._renderedComponent, transaction, this._nativeParent, this._nativeContainerInfo, this._processChildContext(context));

      // 将HTML插入DOM中
      this._replaceNodeWithMarkup(oldNativeNode, nextMarkup, prevComponentInstance);
    }
},
    
_renderValidatedComponent: function() {
    var renderedComponent;
    ReactCurrentOwner.current = this;
    try {
      renderedComponent =
        this._renderValidatedComponentWithoutOwnerOrContext();
    } finally {
      ReactCurrentOwner.current = null;
    }

    return renderedComponent;
},

_renderValidatedComponentWithoutOwnerOrContext: function() {
    var inst = this._instance;
    // 看到render方法了把，应该放心了把~
    var renderedComponent = inst.render();

    return renderedComponent;
},

```
和mountComponent中一样，updateComponent也是用递归的方式将各子组件进行update的。这里要特别注意的是DOM diff。DOM diff是React中渲染加速的关键所在，它会帮我们算出virtual DOM中真正变化的部分，并对这部分进行原生DOM操作。为了避免循环递归对比节点的低效率，React中做了假设，即只对层级不变，type不变，key不变的组件进行Virtual DOM更新。这其中的关键是shouldUpdateReactComponent，下面分析

```javascript
function shouldUpdateReactComponent(prevElement, nextElement) {
  var prevEmpty = prevElement === null || prevElement === false;
  var nextEmpty = nextElement === null || nextElement === false;
  if (prevEmpty || nextEmpty) {
    return prevEmpty === nextEmpty;
  }

  var prevType = typeof prevElement;
  var nextType = typeof nextElement;
  // React DOM diff算法
  // 如果前后两次为数字或者字符,则认为只需要update(处理文本元素)
  // 如果前后两次为DOM元素或React元素,则必须在同一层级内,且type和key不变(key用于listView等组件,很多时候我们没有设置type)才update,否则先unmount再重新mount
  if (prevType === 'string' || prevType === 'number') {
    return (nextType === 'string' || nextType === 'number');
  } else {
    return (
      nextType === 'object' &&
      prevElement.type === nextElement.type &&
      prevElement.key === nextElement.key
    );
  }
}
```

# 销毁
前面提到过，更新组件时，如果不满足DOM diff条件，会先unmountComponent, 然后再mountComponent，下面我们来分析下unmountComponent时都发生了什么事。和mountComponent的多态一样，不同type的ReactComponent也会有不同的unmountComponent行为。我们来分析下React自定义组件，也就是ReactCompositeComponent中的unmountComponent。

```javascript
 unmountComponent: function(safely) {
    if (!this._renderedComponent) {
      return;
    }
    var inst = this._instance;

    // 调用componentWillUnmount
    if (inst.componentWillUnmount && !inst._calledComponentWillUnmount) {
      inst._calledComponentWillUnmount = true;
      // 安全模式下，将componentWillUnmount包在try-catch中。否则直接componentWillUnmount
      if (safely) {
        var name = this.getName() + '.componentWillUnmount()';
        ReactErrorUtils.invokeGuardedCallback(name, inst.componentWillUnmount.bind(inst));
      } else {
        inst.componentWillUnmount();
      }
    }

    // 递归调用unMountComponent来销毁子组件
    if (this._renderedComponent) {
      ReactReconciler.unmountComponent(this._renderedComponent, safely);
      this._renderedNodeType = null;
      this._renderedComponent = null;
      this._instance = null;
    }

    // reset等待队列和其他等待状态
    this._pendingStateQueue = null;
    this._pendingReplaceState = false;
    this._pendingForceUpdate = false;
    this._pendingCallbacks = null;
    this._pendingElement = null;

    // reset内部变量,防止内存泄漏
    this._context = null;
    this._rootNodeID = null;
    this._topLevelWrapper = null;

    // 将组件从map中移除,还记得我们在mountComponent中将它加入了map中的吧
    ReactInstanceMap.remove(inst);
  },

```

> 可见，unmountComponent还是比较简单的，它就做三件事
1.调用componentWillUnmount()
2.递归调用unmountComponent(),销毁子组件
3.将内部变量置空，防止内存泄漏

# 总结
React自定义组件创建期，存在期，销毁期三个阶段的生命周期调用上面都讲完了。三个入口函数mountComponent，updateComponent，unmountComponent尤其关键。大家如果有兴趣，还可以自行分析ReactDOMEmptyComponent，ReactDOMComponent和ReactDOMTextComponent的这三个方法。


深入学习React生命周期源码可以帮我们理清各个方法的调用顺序，明白它们都是什么时候被调用的，哪些条件下才会被调用等等。阅读源码虽然有点枯燥，但能够大大加深对上层API接口的理解，并体会设计者设计这些API的良苦用心。

# [React组件生命周期过程说明](http://react-china.org/t/react/1740) 
实例化
首次实例化

getDefaultProps
getInitialState
componentWillMount
render
componentDidMount
实例化完成后的更新

getInitialState
componentWillMount
render
componentDidMount
存在期
组件已存在时的状态改变

componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate
销毁&清理期
componentWillUnmount
说明
生命周期共提供了10个不同的API。

1.getDefaultProps
作用于组件类，只调用一次，返回对象用于设置默认的props，对于引用值，会在实例中共享。

2.getInitialState
作用于组件的实例，在实例创建时调用一次，用于初始化每个实例的state，此时可以访问this.props。

3.componentWillMount
在完成首次渲染之前调用，此时仍可以修改组件的state。

4.render
必选的方法，创建虚拟DOM，该方法具有特殊的规则：

只能通过this.props和this.state访问数据
可以返回null、false或任何React组件
只能出现一个顶级组件（不能返回数组）
不能改变组件的状态
不能修改DOM的输出
5.componentDidMount
真实的DOM被渲染出来后调用，在该方法中可通过this.getDOMNode()访问到真实的DOM元素。此时已可以使用其他类库来操作这个DOM。

在服务端中，该方法不会被调用。

6.componentWillReceiveProps
组件接收到新的props时调用，并将其作为参数nextProps使用，此时可以更改组件props及state。

    componentWillReceiveProps: function(nextProps) {
        if (nextProps.bool) {
            this.setState({
                bool: true
            });
        }
    }
7.shouldComponentUpdate
组件是否应当渲染新的props或state，返回false表示跳过后续的生命周期方法，通常不需要使用以避免出现bug。在出现应用的瓶颈时，可通过该方法进行适当的优化。

在首次渲染期间或者调用了forceUpdate方法后，该方法不会被调用

8.componentWillUpdate
接收到新的props或者state后，进行渲染之前调用，此时不允许更新props或state。

9.componentDidUpdate
完成渲染新的props或者state后调用，此时可以访问到新的DOM元素。

10.componentWillUnmount
组件被移除之前被调用，可以用于做一些清理工作，在componentDidMount方法中添加的所有任务都需要在该方法中撤销，比如创建的定时器或添加的事件监听器。

[原文地址](https://yq.aliyun.com/articles/72330?spm=a2c4e.11153940.blogcont72329.20.38732a71NEGBnM)


