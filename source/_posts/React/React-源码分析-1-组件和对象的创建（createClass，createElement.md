---
title: React 源码分析 1 组件和对象的创建（createClass，createElement)
date: 2018-03-25 21:26:48
tags: "React"
categories: "React"
---

React受大家欢迎的一个重要原因就是可以自定义组件。这样的一方面可以复用开发好的组件，实现一处开发，处处调用，另外也能使用别人开发好的组件，提高封装性。另一方面使得代码结构很清晰，组件间耦合减少，方便维护。ES5创建组件时，调用React.createClass()即可. ES6中使用class myComponent extends React.Component, 其实内部还是调用createClass创建组件。

组件创建我们可以简单类比为Java中ClassLoader加载class。下面来分析下createClass的源码，我们省去了开发阶段错误提示的相关代码，如propType的检查。（if ("development" !== 'production') {}代码段都不进行分析了，这些只在开发调试阶段调用）

# 组件创建
```javascript
createClass: function (spec) {
    var Constructor = function (props, context, updater) {
      // 触发自动绑定
      if (this.__reactAutoBindPairs.length) {
        bindAutoBindMethods(this);
      }

      // 初始化参数
      this.props = props;
      this.context = context;
      this.refs = emptyObject;  // 本组件对象的引用，可以利用它来调用组件的方法
      this.updater = updater || ReactNoopUpdateQueue;

      // 调用getInitialState()来初始化state变量
      this.state = null;
      var initialState = this.getInitialState ? this.getInitialState() : null;
      this.state = initialState;
    };

    // 继承父类
    Constructor.prototype = new ReactClassComponent();
    Constructor.prototype.constructor = Constructor;
    Constructor.prototype.__reactAutoBindPairs = [];

    injectedMixins.forEach(mixSpecIntoComponent.bind(null, Constructor));

    mixSpecIntoComponent(Constructor, spec);

    // 调用getDefaultProps，并挂载到组件类上。defaultProps是类变量，使用ES6写法时更清晰
    if (Constructor.getDefaultProps) {
      Constructor.defaultProps = Constructor.getDefaultProps();
    }

    // React中暴露给应用调用的方法，如render componentWillMount。
    // 如果应用未设置，则将他们设为null
    for (var methodName in ReactClassInterface) {
      if (!Constructor.prototype[methodName]) {
        Constructor.prototype[methodName] = null;
      }
    }

    return Constructor;
  },

```

<!-- more -->
> createClass主要做的事情有
1.定义构造方法Constructor，构造方法中进行props，refs等的初始化，并调用getInitialState来初始化state
2.调用getDefaultProps，并放在defaultProps类变量上。这个变量不属于某个单独的对象。可理解为static 变量
3.将React中暴露给应用，但应用中没有设置的方法，设置为null。

# 对象的创建

JSX中创建React元素最终会被babel转译为createElement(type, config, children), babel根据JSX中标签的首字母来判断是原生DOM组件，还是自定义React组件。如果首字母大写，则为React组件。这也是为什么ES6中React组件类名必须大写的原因。如下面代码

```javascript
<div className="title" ref="example">
    <span>123</span>    // 原生DOM组件，首字母小写
    <ErrorPage title='123456' desc={[]}/>    // 自定义组件，首字母大写
</div>
```
转译完成后
```javascript
React.createElement(
        'div',    // type,标签名,原生DOM对象为String
        {
            className: 'title',
            ref: 'example'
        },   // config，属性
        React.createElement('span', null, '123'),   // children，子元素
        React.createElement(
            // type,标签名,React自定义组件的type不为String.
            // _errorPage2.default为从其他文件中引入的React组件
            _errorPage2.default,    
            {
                title: '123456',
                desc: []
            }
        )   // children，子元素
)

```

下面来分析下createElement的源码

```javascript
ReactElement.createElement = function (type, config, children) {
  var propName;

  // 初始化参数
  var props = {};

  var key = null;
  var ref = null;
  var self = null;
  var source = null;

  // 从config中提取出内容，如ref key props
  if (config != null) {
    ref = config.ref === undefined ? null : config.ref;
    key = config.key === undefined ? null : '' + config.key;
    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;

    // 提取出config中的prop，放入props变量中
    for (propName in config) {
      if (config.hasOwnProperty(propName) && !RESERVED_PROPS.hasOwnProperty(propName)) {
        props[propName] = config[propName];
      }
    }
  }

  // 处理children，挂到props的children属性下
  // 入参的前两个为type和config，后面的就都是children参数了。故需要减2
  var childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    // 只有一个参数时，直接挂到children属性下，不是array的方式
    props.children = children;
  } else if (childrenLength > 1) {
    // 不止一个时，放到array中，然后将array挂到children属性下
    var childArray = Array(childrenLength);
    for (var i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    props.children = childArray;
  }

  // 取出组件类中的静态变量defaultProps，并给未在JSX中设置值的属性设置默认值
  if (type && type.defaultProps) {
    var defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }

  // 返回一个ReactElement对象
  return ReactElement(type, key, ref, self, source, ReactCurrentOwner.current, props);
};
```

下面来看ReactElement源码

```javascript
var ReactElement = function (type, key, ref, self, source, owner, props) {
  var element = {
    // This tag allow us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // ReactElement对象上的四个变量，特别关键
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner
  };
    return element;
}
```
可以看到仅仅是给ReactElement对象内的成员变量赋值而已，不在赘述。

流程如如下

# 组件对象初始化
在mountComponent()挂载组件中，会进行组件渲染，调用到instantiateReactComponent()方法。这个过程我们在React生命周期方法中再详细讲述，这里有个大体了解即可。instantiateReactComponent()根据ReactElement中不同的type字段，创建不同类型的组件对象。源码如下

```javascript
// 初始化组件对象，node是一个ReactElement对象，是节点元素在React中的表示
function instantiateReactComponent(node) {
  var instance;

  var isEmpty = node === null || node === false;
  if (isEmpty) {
    // 空对象
    instance = ReactEmptyComponent.create(instantiateReactComponent);
  } else if (typeof node === 'object') {
    // 组件对象，包括DOM原生的和React自定义组件
    var element = node;

    // 根据ReactElement中的type字段区分
    if (typeof element.type === 'string') {
      // type为string则表示DOM原生对象，比如div span等。可以参看上面babel转译的那段代码
      instance = ReactNativeComponent.createInternalComponent(element);
    } else if (isInternalComponentType(element.type)) {
      // 保留给以后版本使用，此处暂时不会涉及到
      instance = new element.type(element);
    } else {
      // React自定义组件
      instance = new ReactCompositeComponentWrapper(element);
    }
  } else if (typeof node === 'string' || typeof node === 'number') {
    // 元素是一个string时，对应的比如<span>123</span> 中的123
    // 本质上它不是一个ReactElement，但为了统一，也按照同样流程处理，称为ReactDOMTextComponent
    instance = ReactNativeComponent.createInstanceForText(node);
  } else {
    // 无处理
  }

  // 初始化参数，这两个参数是DOM diff时用到的
  instance._mountIndex = 0;
  instance._mountImage = null;

  return instance;
}
```

>故我们可以看到有四种创建组件元素的方式，同时对应四种ReactElement
1.ReactEmptyComponent.create(), 创建空对象ReactDOMEmptyComponent
2.ReactNativeComponent.createInternalComponent(), 创建DOM原生对象 ReactDOMComponent
3.new ReactCompositeComponentWrapper(), 创建React自定义对象ReactCompositeComponent
4.ReactNativeComponent.createInstanceForText(), 创建文本对象 ReactDOMTextComponent

下面分别分析这几种对象，和创建他们的过程。

**ReactDOMEmptyComponent**
由ReactEmptyComponent.create()创建，最终生成ReactDOMEmptyComponent对象，源码如下

```javascript
var emptyComponentFactory;

var ReactEmptyComponentInjection = {
  injectEmptyComponentFactory: function (factory) {
    emptyComponentFactory = factory;
  }
};

var ReactEmptyComponent = {
  create: function (instantiate) {
    return emptyComponentFactory(instantiate);
  }
};

ReactEmptyComponent.injection = ReactEmptyComponentInjection;

ReactInjection.EmptyComponent.injectEmptyComponentFactory(function (instantiate) {
  // 前面比较绕，关键就是这句话，创建ReactDOMEmptyComponent对象
   return new ReactDOMEmptyComponent(instantiate);
});

// 各种null，就不分析了
var ReactDOMEmptyComponent = function (instantiate) {
  this._currentElement = null;
  this._nativeNode = null;
  this._nativeParent = null;
  this._nativeContainerInfo = null;
  this._domID = null;
};
```

**ReactDOMComponent**
由ReactNativeComponent.createInternalComponent()创建。这里注意原生组件不代表是DOM组件，而是React封装过的Virtual DOM对象。React并不直接操作原生DOM。

大家可以自己看ReactDOMComponent的源码。重点看下ReactDOMComponent.Mixin
```javascript
ReactDOMComponent.Mixin = {
  mountComponent: function (transaction, nativeParent, nativeContainerInfo, context) {},
  _createOpenTagMarkupAndPutListeners: function (transaction, props){},
  _createContentMarkup: function (transaction, props, context) {},
  _createInitialChildren: function (transaction, props, context, lazyTree) {}
  receiveComponent: function (nextElement, transaction, context) {},
  updateComponent: function (transaction, prevElement, nextElement, context) {},
  _updateDOMProperties: function (lastProps, nextProps, transaction) {},
  _updateDOMChildren: function (lastProps, nextProps, transaction, context) {},
  getNativeNode: function () {},
  unmountComponent: function (safely) {},
  getPublicInstance: function () {}
}
```

其中暴露给外部的比较关键的是mountComponent，receiveComponen， updateComponent，unmountComponent。他们会引发React生命周期方法的调用，下一节再讲。

**ReactCompositeComponent**
由new ReactCompositeComponentWrapper()创建，重点看下ReactCompositeComponentMixin

```javascript
var ReactCompositeComponentMixin = {
  // new对应的方法，创建ReactCompositeComponent对象
  construct: function(element) {},
  mountComponent,   // 初始挂载组件时被调用，仅一次
  performInitialMountWithErrorHandling, // 和performInitialMount相近，只是多了错误处理
  performInitialMount,  // 执行mountComponent的渲染阶段，会调用到instantiateReactComponent，从而进入初始化React组件的入口
  getNativeNode,
  unmountComponent, // 卸载组件，内存释放等工作
  receiveComponent,
  performUpdateIfNecessary,
  updateComponent,  // setState后被调用，重新渲染组件
  attachRef,    // 将ref指向组件对象，这样我们就可以利用它调用对象内的方法了
  detachRef,    // 将组件的引用从全局对象refs中删掉，这样我们就不能利用ref找到组件对象了
  instantiateReactComponent，  // 初始化React组件的入口，在mountComponent时的渲染阶段会被调用
}

```

**ReactDOMTextComponent**

由ReactNativeComponent.createInstanceForText()创建，我们也不细细分析了，主要入口代码如下，大家可以自行分析。

```javascript
var ReactDOMTextComponent = function (text) {
  this._currentElement = text;
  this._stringText = '' + text;
};

_assign(ReactDOMTextComponent.prototype, {
  mountComponent,
  receiveComponent,
  getNativeNode,
  unmountComponent
}

```

[原文地址](https://zhuanlan.zhihu.com/p/25881952)