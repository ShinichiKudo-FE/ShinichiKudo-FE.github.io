---
title: React V16.3 即将更改的生命周期
date: 2018-04-30 22:30:04
tags: "React"
categories: "React"
---
# 即将更改的生命周期

一年多来，React团队一直致力于实现异步渲染。上个月，他在JSConf冰岛的演讲中，丹揭示了一些令人兴奋的新的异步渲染可能性。现在，我们希望与您分享我们在学习这些功能时学到的一些经验教训，以及一些帮助您准备组件以在启动时进行异步渲染的方法。

我们了解到的最大问题之一是，我们的一些传统组件生命周期会导致一些不安全的编码实践。他们是：

* componentWillMount
* componentWillReceiveProps
* componentWillUpdate

这些生命周期方法经常被误解和滥用;此外，我们预计他们的潜在滥用可能在异步渲染方面有更大的问题。因此，我们将在即将发布的版本中为这些生命周期添加一个“`UNSAFE_`”前缀。 （这里，“不安全”不是指安全性，而是表示使用这些生命周期的代码将更有可能在未来的React版本中存在缺陷，特别是一旦启用了异步渲染）。

## 逐步迁移路径

React遵循语义版本控制, 所以这种改变将是渐进的。我们目前的计划是：

* **16.3**：为不安全生命周期引入别名UNSAFE_componentWillMount，UNSAFE_componentWillReceiveProps和UNSAFE_componentWillUpdate。 （旧的生命周期名称和新的别名都可以在此版本中使用。）

* **未来的16.x版本**：为componentWillMount，componentWillReceiveProps和componentWillUpdate启用弃用警告。 （旧的生命周期名称和新的别名都可以在此版本中使用，但旧名称会记录DEV模式警告。）

* **17.0**：删除componentWillMount，componentWillReceiveProps和componentWillUpdate。 （从现在开始，只有新的“UNSAFE_”生命周期名称将起作用。）

**请注意，如果您是React应用程序开发人员，那么您不必对遗留方法进行任何操作。即将发布的16.3版本的主要目的是让开源项目维护人员在任何弃用警告之前更新其库。这些警告将在未来的16.x版本发布之前不会启用**。

## 从传统生命周期迁移

如果您想开始使用React 16.3中引入的新组件API（或者如果您是维护人员提前更新库），以下是一些示例，我们希望这些示例可以帮助您开始考虑组件的变化。随着时间的推移，我们计划在文档中添加额外的“配方”，以展示如何以避免有问题的生命周期的方式执行常见任务。

在开始之前，我们将简要概述为16.3版计划的生命周期更改：

* 我们正在添加以下生命周期别名：

(1) UNSAFE_componentWillMount，

(2) UNSAFE_componentWillReceiveProps

(3) UNSAFE_componentWillUpdate。 （旧的生命周期名称和新的别名都将受支持。）

* 我们介绍了两个新的生命周期，分别是getDerivedStateFromProps和getSnapshotBeforeUpdate。

### 新的生命周期: getDerivedStateFromProps

<!-- more -->

```js

class Example extends React.Component {

  static getDerivedStateFromProps(nextProps, prevState) {

    // ...

  }

}
```

新的静态`getDerivedStateFromProps`生命周期在组件实例化以及接收新`props`后调用。它可以返回一个对象来更新`state`，或者返回null来表示新的`props`不需要任何`state`更新。

与`componentDidUpdate`一起，这个新的生命周期应该覆盖传统`componentWillReceiveProps`的所有用例。

### 新的生命周期: getSnapshotBeforeUpdate

```js
class Example extends React.Component {

  getSnapshotBeforeUpdate(prevProps, prevState) {

    // ...

  }

}
```

新的`getSnapshotBeforeUpdate`生命周期在更新之前被调用（例如，在DOM被更新之前）。此生命周期的返回值将作为第三个参数传递给`componentDidUpdate`。 （这个生命周期不是经常需要的，但可以用于在恢复期间手动保存滚动位置的情况。）

You can find their type signatures in this gist.

我们看看如何在使用这两种生命周期的，例子如下:

#### 例如

* Initializing state（初始化状态）Ini

* Fetching external data（获取外部数据）

* Adding event listeners（添加事件监听）

* Updating state based on props（基于props更新state）

* Invoking external callbacks(调用外部的callbacks)

* Side effects on props change

* Fetching external data when props change（props改变时获取外部数据）

* Reading DOM properties before an update(在更新之前读取DOM属性)

> 注意
为简洁起见，下面的示例是使用实验类属性转换编写的，但如果没有它，则应用相同的迁移策略。

### 初始化状态：

这个例子展示了一个调用`componentWillMount`中带有`setState`的组件：

```js
// Before

class ExampleComponent extends React.Component {

  state = {};

  componentWillMount() {

    this.setState({

      currentColor: this.props.defaultColor,

      palette: 'rgb',

    });

  }

}
```

这种类型的组件最简单的重构是将状态初始化移动到构造函数或属性初始值设定项，如下所示：

```js
// After

class ExampleComponent extends React.Component {

  state = {

    currentColor: this.props.defaultColor,

    palette: 'rgb',

  };

}
```

### 获取外部数据

以下是使用componentWillMount获取外部数据的组件示例：

```js
// Before

class ExampleComponent extends React.Component {

  state = {

    externalData: null,

  };



  componentWillMount() {

    this._asyncRequest = asyncLoadData().then(

      externalData => {

        this._asyncRequest = null;

        this.setState({externalData});

      }

    );

  }



  componentWillUnmount() {

    if (this._asyncRequest) {

      this._asyncRequest.cancel();

    }

  }

  render() {

    if (this.state.externalData === null) {

      // Render loading state ...

    } else {

      // Render real UI ...

    }

  }

}

```

对于大多数用例，建议的升级路径是将数据提取移入`componentDidMount`：

```js
// After

class ExampleComponent extends React.Component {

  state = {

    externalData: null,

  };
  componentDidMount() {

    this._asyncRequest = asyncLoadData().then(

      externalData => {

        this._asyncRequest = null;

        this.setState({externalData});

      }

    );

  }

  componentWillUnmount() {

    if (this._asyncRequest) {

      this._asyncRequest.cancel();

    }

  }

  render() {

    if (this.state.externalData === null) {

      // Render loading state ...

    } else {

      // Render real UI ...

    }

  }

}
```

有一个常见的错误观念认为，在`componentWillMount`中提取可以避免第一个空的渲染。在实践中，这从来都不是真的，因为React总是在`componentWillMount`之后立即执行渲染。如果数据在`componentWillMount`触发的时间内不可用，则无论你在哪里提取数据，第一个渲染仍将显示加载状态。这就是为什么在绝大多数情况下将提取移到`componentDidMount`没有明显效果。

> 注意：
一些高级用例（例如，像Relay这样的库）可能想要尝试使用热切的预取异步数据。在这里可以找到一个这样做的例子。

当支持服务器渲染时，目前需要同步提供数据 - `componentWillMount`通常用于此目的，但构造函数可以用作替换。即将到来的悬念API将使得异步数据在客户端和服务器呈现中都可以清晰地获取。

### 添加时间监听

下面是一个在安装时监听外部事件调度程序的组件示例：

```js
// Before

class ExampleComponent extends React.Component {

  componentWillMount() {

    this.setState({

      subscribedValue: this.props.dataSource.value,

    });



    // This is not safe; it can leak!

    this.props.dataSource.subscribe(

      this.handleSubscriptionChange

    );

  }


  componentWillUnmount() {

    this.props.dataSource.unsubscribe(

      this.handleSubscriptionChange

    );

  }

  handleSubscriptionChange = dataSource => {

    this.setState({

      subscribedValue: dataSource.value,

    });

  };

}
```

不幸的是，这会导致服务器渲染（`componentWillUnmount`永远不会被调用）和异步渲染（在渲染完成之前渲染可能被中断，导致`componentWillUnmount`不被调用）的内存泄漏。

人们经常认为`componentWillMount`和`componentWillUnmount`总是配对，但这并不能保证。只有调用`componentDidMount`后，React才能保证稍后调用`componentWillUnmount`进行清理。

出于这个原因，添加事件监听的推荐方式是使用`componentDidMount`生命周期：

```js
// After

class ExampleComponent extends React.Component {

  state = {

    subscribedValue: this.props.dataSource.value,

  };



  componentDidMount() {

    // Event listeners are only safe to add after mount,

    // So they won't leak if mount is interrupted or errors.

    this.props.dataSource.subscribe(

      this.handleSubscriptionChange

    );



    // External values could change between render and mount,

    // In some cases it may be important to handle this case.

    if (

      this.state.subscribedValue !==

      this.props.dataSource.value

    ) {

      this.setState({

        subscribedValue: this.props.dataSource.value,

      });

    }

  }



  componentWillUnmount() {

    this.props.dataSource.unsubscribe(

      this.handleSubscriptionChange

    );

  }



  handleSubscriptionChange = dataSource => {

    this.setState({

      subscribedValue: dataSource.value,

    });

  };

}
```

有时候更新监听以响应属性变化很重要。如果您使用的是像Redux或MobX这样的库，库的容器组件会为您处理。对于应用程序作者，我们创建了一个小型库`create-subscription`来帮助解决这个问题。我们会将它与React 16.3一起发布。

我们可以使用`create-subscription`来传递监听的值，而不是像上例那样传递监听 的`dataSource prop`。

```js
import {createSubscription} from 'create-subscription';



const Subscription = createSubscription({

  getCurrentValue(sourceProp) {

    // Return the current value of the subscription (sourceProp).

    return sourceProp.value;

  },



  subscribe(sourceProp, callback) {

    function handleSubscriptionChange() {

      callback(sourceProp.value);

    }



    // Subscribe (e.g. add an event listener) to the subscription (sourceProp).

    // Call callback(newValue) whenever a subscription changes.

    sourceProp.subscribe(handleSubscriptionChange);



    // Return an unsubscribe method.

    return function unsubscribe() {

      sourceProp.unsubscribe(handleSubscriptionChange);

    };

  },

});



// Rather than passing the subscribable source to our ExampleComponent,

// We could just pass the subscribed value directly:

`<Subscription source={dataSource}>`

  {value => `<ExampleComponent subscribedValue={value} />`}

`</Subscription>`;
```

> 注意
像Relay / Apollo这样的库应该使用与创建订阅相同的技术手动管理订阅（如此处所引用的），并采用最适合其库使用的优化方式。

### 基于props更新state

以下是使用旧版`componentWillReceiveProps`生命周期基于新的道具值更新状态的组件示例：

```js
// Before

class ExampleComponent extends React.Component {

  state = {

    isScrollingDown: false,

  };



  componentWillReceiveProps(nextProps) {

    if (this.props.currentRow !== nextProps.currentRow) {

      this.setState({

        isScrollingDown:

          nextProps.currentRow > this.props.currentRow,

      });

    }

  }

}
```

尽管上面的代码本身并没有问题，但`componentWillReceiveProps`生命周期通常会被错误地用于解决问题。因此，该方法将被弃用。

从版本16.3开始，`更新state以响应props`更改的推荐方法是使用新的静态`getDerivedStateFromProps`生命周期。 （生命周期在组件创建时以及每次收到新道具时调用）：

```js
// After

class ExampleComponent extends React.Component {

  // Initialize state in constructor,

  // Or with a property initializer.

  state = {

    isScrollingDown: false,

    lastRow: null,

  };



  static getDerivedStateFromProps(nextProps, prevState) {

    if (nextProps.currentRow !== prevState.lastRow) {

      return {

        isScrollingDown:

          nextProps.currentRow > prevState.lastRow,

        lastRow: nextProps.currentRow,

      };

    }



    // Return null to indicate no change to state.

    return null;

  }

}
```

你可能会注意到在上面的例子中，`props.currentRow`是一个镜像状态（如state.lastRow）。这使得`getDerivedStateFromProps`可以像在`componentWillReceiveProps`中一样访问以前的props值。

您可能想知道为什么我们不只是将先前的`props`作为参数传递给`getDerivedStateFromProps`。我们在设计API时考虑了这个选项，但最终决定反对它，原因有两个

* 在第一次调用getDerivedStateFromProps（实例化后）时，prevProps参数将为null，需要在访问prevProps时添加if-not-null检查。

* 没有将以前的props传递给这个函数，在未来版本的React中释放内存的一个步骤。 （如果React不需要将先前的道具传递给生命周期，那么它不需要将先前的道具对象保留在内存中。）

> Note
如果您正在编写共享组件，那么react-lifecycles-compat polyfill可以使新的getDerivedStateFromProps生命周期与旧版本的React一起使用。详细了解如何在下面使用它。

### 调用外部回调函数

下面是一个在内部状态发生变化时调用外部函数的组件示例：

```js
// Before

class ExampleComponent extends React.Component {

  componentWillUpdate(nextProps, nextState) {

    if (

      this.state.someStatefulValue !==

      nextState.someStatefulValue

    ) {

      nextProps.onChange(nextState.someStatefulValue);

    }

  }

}
```

在异步模式下使用`componentWillUpdate`都是不安全的，因为外部回调可能会多次调用只更新一次。相反，应该使用`componentDidUpdate`生命周期，因为它保证每次更新只调用一次：

```js
// After

class ExampleComponent extends React.Component {

  componentDidUpdate(prevProps, prevState) {

    if (

      this.state.someStatefulValue !==

      prevState.someStatefulValue

    ) {

      this.props.onChange(this.state.someStatefulValue);

    }

  }

}
```

### props改变的副作用

与上述 事例类似，有时组件在道具更改时会产生副作用。

```js
// Before

class ExampleComponent extends React.Component {

  componentWillReceiveProps(nextProps) {

    if (this.props.isVisible !== nextProps.isVisible) {

      logVisibleChange(nextProps.isVisible);

    }

  }

}
```

与`componentWillUpdate`一样，`componentWillReceiveProps`可能会多次调用但是只更新一次。出于这个原因，避免在此方法中导致的副作用非常重要。相反，应该使用`componentDidUpdate`，因为它保证每次更新只调用一次：

```js
// After

class ExampleComponent extends React.Component {

  componentDidUpdate(prevProps, prevState) {

    if (this.props.isVisible !== prevProps.isVisible) {

      logVisibleChange(this.props.isVisible);

    }

  }

}
```

### props改变时获取外部数据

以下是根据`propsvalues`提取外部数据的组件示例：

```js
// Before

class ExampleComponent extends React.Component {

  state = {

    externalData: null,

  };



  componentDidMount() {

    this._loadAsyncData(this.props.id);

  }



  componentWillReceiveProps(nextProps) {

    if (nextProps.id !== this.props.id) {

      this.setState({externalData: null});

      this._loadAsyncData(nextProps.id);

    }

  }



  componentWillUnmount() {

    if (this._asyncRequest) {

      this._asyncRequest.cancel();

    }

  }



  render() {

    if (this.state.externalData === null) {

      // Render loading state ...

    } else {

      // Render real UI ...

    }

  }



  _loadAsyncData(id) {

    this._asyncRequest = asyncLoadData(id).then(

      externalData => {

        this._asyncRequest = null;

        this.setState({externalData});

      }

    );

  }

}

```

此组件的推荐升级路径是将数据更新移动到`componentDidUpdate`中。在渲染新道具之前，您还可以使用新的`getDerivedStateFromProps`生命周期清除陈旧的数据：

```js
// After

class ExampleComponent extends React.Component {

  state = {

    externalData: null,

  };



  static getDerivedStateFromProps(nextProps, prevState) {

    // Store prevId in state so we can compare when props change.

    // Clear out previously-loaded data (so we don't render stale stuff).

    if (nextProps.id !== prevState.prevId) {

      return {

        externalData: null,

        prevId: nextProps.id,

      };

    }



    // No state update necessary

    return null;

  }



  componentDidMount() {

    this._loadAsyncData(this.props.id);

  }



  componentDidUpdate(prevProps, prevState) {

    if (this.state.externalData === null) {

      this._loadAsyncData(this.props.id);

    }

  }



  componentWillUnmount() {

    if (this._asyncRequest) {

      this._asyncRequest.cancel();

    }

  }



  render() {

    if (this.state.externalData === null) {

      // Render loading state ...

    } else {

      // Render real UI ...

    }

  }



  _loadAsyncData(id) {

    this._asyncRequest = asyncLoadData(id).then(

      externalData => {

        this._asyncRequest = null;

        this.setState({externalData});

      }

    );

  }

}

```

> 注意
如果您使用支持取消的HTTP库（如axios），那么卸载时取消正在进行的请求很简单。对于原生Promise，您可以使用如下所示的方法。

### 在更新之前读取DOM属性

下面是一个组件的例子，它在更新之前从DOM中读取属性，以便在列表中保持滚动位置：

```js
componentWillUpdate(nextProps, nextState) {

    // Are we adding new items to the list?

    // Capture the scroll position so we can adjust scroll later.

    if (this.props.list.length < nextProps.list.length) {

      this.previousScrollOffset =

        this.listRef.scrollHeight - this.listRef.scrollTop;

    }

  }



  componentDidUpdate(prevProps, prevState) {

    // If previousScrollOffset is set, we've just added new items.

    // Adjust scroll so these new items don't push the old ones out of view.

    if (this.previousScrollOffset !== null) {

      this.listRef.scrollTop =

        this.listRef.scrollHeight -

        this.previousScrollOffset;

      this.previousScrollOffset = null;

    }

  }

  render() {

    return (

      `<div>`

        {/* ...contents... */}

      `</div>`

    );

  }



  setListRef = ref => {

    this.listRef = ref;

  };

}
```

在上面的例子中，`componentWillUpdate`被用来读取DOM属性。但是，对于异步渲染，“render”阶段生命周期（如c`omponentWillUpdate`和`render`）与“commit”阶段生命周期（如`componentDidUpdate`）之间可能存在延迟。如果用户在这段时间内做了类似调整窗口大小的操作，则从`componentWillUpdate`中读取的`scrollHeight`值将失效。

解决此问题的方法是使用新的“commit”阶段生命周期`getSnapshotBeforeUpdate`。在数据发生变化之前立即调用该方法（例如，在更新DOM之前）。它可以将React的值作为参数传递给`componentDidUpdate`，在数据发生变化后立即调用它。

这两个生命周期可以像这样一起使用：

```js
class ScrollingList extends React.Component {

  listRef = null;



  getSnapshotBeforeUpdate(prevProps, prevState) {

    // Are we adding new items to the list?

    // Capture the scroll position so we can adjust scroll later.

    if (prevProps.list.length < this.props.list.length) {

      return (

        this.listRef.scrollHeight - this.listRef.scrollTop

      );

    }

    return null;

  }



  componentDidUpdate(prevProps, prevState, snapshot) {

    // If we have a snapshot value, we've just added new items.

    // Adjust scroll so these new items don't push the old ones out of view.

    // (snapshot here is the value returned from getSnapshotBeforeUpdate)

    if (snapshot !== null) {

      this.listRef.scrollTop =

        this.listRef.scrollHeight - snapshot;

    }

  }



  render() {

    return (

      `<div>`

        {/* ...contents... */}

      `</div>`

    );

  }



  setListRef = ref => {

    this.listRef = ref;

  };

}
```

> 注意
如果您正在编写共享组件，那么`react-lifecycles-compat polyfill`可以使新的`getSnapshotBeforeUpdate`生命周期与旧版本的React一起使用。详细了解如何使用它。

## 其它情况

除了以上的一些常见的例子，还可能会有别的情况本篇文章没有涵盖到，如果您以本博文未涉及的方式使用除了以上 `componentWillMount`，`componentWillUpdate`或`componentWillReceiveProps`，并且不确定如何迁移这些传统生命周期，你可以提供您的代码示例和我们的文档，并且一起提交一个新问题。我们将在更新这份文件时提供新的替代模式。

当React 16.3发布时，我们还将发布一个新的npm包， react-lifecycles-compat。该npm包会填充组件，以便新的getDerivedStateFromProps和getSnapshotBeforeUpdate生命周期也可以与旧版本的React（0.14.9+）一起使用。

要使用这个polyfill，首先将它作为依赖项添加到您的库中：

```js
# Yarn

yarn add react-lifecycles-compat

# NPM

npm install react-lifecycles-compat --save
```

接下来，更新您的组件以使用新的生命周期（如上所述）。

最后，使用polyfill将组件向后兼容旧版本的React：

```js
import React from 'react';

import {polyfill} from 'react-lifecycles-compat';



class ExampleComponent extends React.Component {

  static getDerivedStateFromProps(nextProps, prevState) {

    // Your state update logic here ...

  }

}

// Polyfill your component to work with older versions of React:

polyfill(ExampleComponent);

export default ExampleComponent;
```

[原文地址](https://mp.weixin.qq.com/s/dYVdOGtcka3_ODfap1pvJQ)