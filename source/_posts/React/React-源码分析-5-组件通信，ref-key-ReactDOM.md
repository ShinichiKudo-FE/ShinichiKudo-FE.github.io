---
title: 'React源码分析 5 组件通信，ref,key,ReactDOM'
date: 2018-03-27 11:16:58
tags: "React"
categories: "React"
---

# 组件之间通信

## 父组件向子组件通信
React规定了明确的单向数据流，利用props将数据从父组件传递给子组件。故我们可以利用props，让父组件给子组件通信。故父组件向子组件通信还是很容易实现的。引申一点，父组件怎么向孙子组件通信呢？可以利用props进行层层传递，使用ES6的...运算符可以用很简洁的方式把props传递给孙子组件。这里我们就不举例了。

要注意的一点是，setProps,replaceProps两个API已经被废弃了，React建议我们在顶层使用ReactDOM.reader()进行props更新。

## 子组件向父组件通信
React数据流是单向的，只能从父组件传递到子组件。那么子组件怎么向父组件通信呢？其实仍然可以利用props。父组件利用props传递方法给子组件，子组件回调这个方法的同时，将数据传递进去，使得父组件的相关方法得到回调，这个时候就可以把数据从子组件传递给父组件了。看一个例子。

```javascript
class Parent extends React.Component {
  handleChildMsg(msg) {
    // 父组件处理消息
    console.log("parent: " + msg);
  }
  
  render() {
    return (
      <div>
          <Child transferMsg = {msg => this.handleChildMsg(msg)} />
      </div>
    );
  }
}

class Child extends React.Component {
  componentDidMount() {
    // 子组件中调用父组件的方法，将数据以参数的方式传递给父组件，这样父组件方法就得到回调了，也收到数据了
    this.props.transferMsg("child has mounted");
  }
  
  render() {
    return (
      <div>child</div>
    )
  }
}
```
<!-- more -->

这个例子应该很清楚了，通过回调的方式，可以将数据从子组件传递给父组件。引申一下，孙子组件怎么把数据传递给父组件呢？同样可以利用props层层回调。利用ES6的**...运算符**也可以用比较简洁的方式完成props层层回调。

## 兄弟组件通信 - 发布/订阅

兄弟组件可以利用父组件进行中转，将数据先由child1传给parent，然后parent传给child2. 这个方法显然耦合比较严重，传递次数过多，容易引发父组件不必要的生命周期回调，甚至影响其他子组件，故强烈建议不要使用这个方式。

我们可以利用观察者模式来解决这个问题。观察者模式采用发布/订阅的方法，可以将消息发送者和接收者完美解耦。React中可以引入eventProxy模块，利用eventProxy.trigger()方法发布消息，eventProxy.on()方法监听并接收消息。eventProxy我们就不展开讲了。下面看一个例子

```javascript
import eventProxy from '../eventProxy'

class Child1 extends React.Component {
  componentDidMount() {
    // 发布者，发出消息
    eventProxy.trigger('msg', 'child1 has been mounted');
  }
  render() {
    return (
      <div>child1</div>
    );
  }
}

class Child2 extends React.Component {
  componentDidMount() {
    // 订阅者，监听并接收消息
    eventProxy.on('msg', (msg) => {console.log('msg: ' + msg)});
  }
  
  render() {
    return (
      <div>child2</div>
    );
  } 
}

```

## 嵌套层级深组件 - context
祖父组件和孙子组件通信时，我们有时候还是觉得通过props有点繁琐了。此时可以考虑使用context全局变量。

>使用方法：
1.祖父组件中定义getChildContext()方法，将要传递给孙子的数据放在其中
2.祖父组件中childContextTypes申明要传递的数据类型
3.孙子组件中contextTypes申明可以接收的数据类型
4.孙子组件通过this.context访问祖父传递进来的数据。

采用全局变量的方式，容易导致数据混乱，分不清数据是从哪儿来的，不容易控制。建议少用这种方式。

## Redux
还可以利用Flux和Redux架构来进行组件通信，这个我们以后再专门详细分析。

# refs

## 用法
我们在getRender()返回的JSX中，可以在标签中加入ref属性，然后通过refs.ref就可以访问到我们的Component了，例如。

```javascript

class Parent extends React.Component {
  getRender() {
    <div>
      <Child ref = 'child' />
    </div>
  }
  
  componentDidMount() {
    // 通过refs可以拿到子元素,然后就可以访问到子元素的方法了
    let child = this.refs.child;
    child.test();
  }
}

class Child extends React.Component {
  test() {
    console.log("child method called by ref");
  }
}

```

## attachRef 将子组件引用保存到父组件refs对象中

refs的用法很简单，只需要JSX中定义好ref属性即可。那么首先一个问题来了，refs这个对象在哪儿定义的呢？还记得createClass方法的constructor吧，它里面会定义并初始化refs对象。源码如下

```javascript
createClass: function (spec) {
    // 自定义React类的构造方法，通过它创建一个React.Component对象
    var Constructor = identity(function (props, context, updater) {

      // Wire up auto-binding
      if (this.__reactAutoBindPairs.length) {
        bindAutoBindMethods(this);
      }

      this.props = props;
      this.context = context;
      // refs初始化为一个空对象
      this.refs = emptyObject;
      this.updater = updater || ReactNoopUpdateQueue;

      // 调用getInitialState初始化state
      this.state = null;
      var initialState = this.getInitialState ? this.getInitialState() : null;
      this.state = initialState;
    });
    ...
}
```

从上面代码可见，每次创建自定义组件的时候，都会初始化一个为空的refs对象。那么第二个问题来了，ref字符串所指向的对象的引用，是什么时候加入到refs对象中的呢？答案就在ReactCompositeComponent的attachRef方法中，源码如下

```javascript
  attachRef: function(ref, component) {
    // getPublicInstance返回我们的父组件
    var inst = this.getPublicInstance();
    var publicComponentInstance = component.getPublicInstance();
    var refs = inst.refs === emptyObject ? (inst.refs = {}) : inst.refs;
    // 将子元素的引用，以ref属性为key,保存到父元素的refs对象中
    refs[ref] = publicComponentInstance;
  },
```

attachRef方法又是什么时候被调用的呢？我们这儿就不源码分析了。大概说下，mountComponent中，如果element的ref属性不为空，则会以transaction事务的方式调用attachRefs方法，而attachRefs方法中则会调用attachRef方法，将子组件的引用保存到父组件的refs对象中。

## detachRef 从父组件refs对象中删除子组件引用
对内存管理有些了解的同学肯定会有疑惑，既然父组件的refs中保存了子组件引用，那么当子组件被unmountComponent而销毁时，子组件的引用仍然保存在refs对象中，岂不是会导致内存泄漏？React当然不会有这个bug了，秘密就在detachRef方法中，源码如下

```javascript
  detachRef: function(ref) {
    var refs = this.getPublicInstance().refs;
    // 从refs对象中删除key为ref子元素,防止内存泄漏
    delete refs[ref];
  },
```

代码很简单，delete掉ref字符串指向的成员即可。至于detachRef的调用链，我们还得从unmountComponent方法说起。unmountComponent会调用detachRefs方法，而detachRefs中则会调用detachRef，从而将子元素引用从refs中释放掉，防止内存泄漏。也就是说在unmountComponent时，React自动帮我们完成了子元素ref删除，防止内存泄漏。

# key
当我们的子组件是一个数组时，比如类似于Android中的ListView，一个列表中有很多样式一致的项，此时给每个项加上key这个属性就很有作用了。key可以标示当前项的唯一性。

对于数组，其内部包含长度不确定的子项。当组件state变化时，需要重新渲染组件。那么有个问题来了，React是更新组件，还是先销毁再新建组件呢。key就是用来解决这个问题的。如果前后两次key不变，则只需要更新，否则先销毁再更新。

对于子项的key，必须是唯一不重复的。并且尽量传不变的属性，千万不要传无意义的index或者随机值。这样才能尽量以更新的方式来重新渲染。React源码中判断更新方式的源码如下

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

看到key这个属性的重要性了吧。对于数组组件，我们一定要在每个子项上设置一个key，这样可以大大提高DOM diff的性能。


那为什么数组组件之外的其他组件，不用设置key呢？因为他们的type或者在父组件中的位置不同，完全可以区分开，所以不需要key就可以完全确定是哪个组件

# 无状态组件
无状态组件其实本质上就是一个函数，传入props即可，没有state，也没有生命周期方法。组件本身对应的就是render方法。例子如下

```javascript
function Title({color = 'red', text = '标题'}) {
  let style = {
    'color': color
  }
  return (
    <div style = {style}>{text}</div>
  )
}
```
无状态组件不会创建对象，故比较省内存。没有复杂的生命周期方法调用，故流程比较简单。没有state，也不会重复渲染。它本质上就是一个函数而已。

对于没有状态变化的组件，React建议我们使用无状态组件。总之，能用无状态组件的地方，就用无状态组件。

# react DOM
React通过findDOMNode()可以找到组件实例对应的DOM节点，但需要注意的是，我们只能在render()之后，也就是componentDidMount()和componentDidUpdate()中调用。因为只有render后，DOM对象才生成了。

```javascript
class example extends React.Component {
  componentDidMount() {
    // 只有render后才生成了DOM node，才能调用findDOMNode
    let dom = ReactDOM.findDOMNode(this);
  }
}
```


那为什么render后DOM才生成呢，我们可以从源码角度来分析。React源码分析3 — React组件插入DOM流程一文中，我们知道mountComponent解析得到了markup，也就是React组件对应的HTML，会由_mountImageIntoNode方法插入到真实DOM中，故这个事务结束后，才生成了真正的DOM。故肯定只有render之后，才有真实的DOM可以被访问。

那为什么componentDidMount()能访问DOM呢？它不是也在mountComponent()方法流程中吗？这是因为React采用异步事务的方式来调用componentDidMount的，它把componentDidMount放到一个事务队列中，只有当前mountComponent这个事务处理完了，才会回过头去处理componentDidMount，故在componentDidMount中可以拿到真实的DOM。这个设计得给React点赞。这一点可以从源码来分析。

```javascript
mountComponent: function (transaction, nativeParent, nativeContainerInfo, context) {
    // 省略一段代码
    ...
    
    if (inst.componentDidMount) {
      // 调用componentDidMount，以事务的形式。放到queue中，异步的方式，有那么点Android MessageQueue的感觉
      transaction.getReactMountReady().enqueue(inst.componentDidMount, inst);
    }

    return markup;
},
```

另外值得注意的是，React不建议我们碰底层的DOM，因为React有一套性能比较高的DOM diff方式来更新真实DOM。并且容易导致DOM引用忘记释放等内存泄漏问题。一句话，除非不得已，不要碰DOM。

[原文地址](https://yq.aliyun.com/articles/72750?spm=a2c4e.11153940.blogcont72329.29.51012a71kVn38v)