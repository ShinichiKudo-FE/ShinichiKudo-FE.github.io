---
title: react 16.3新特性
date: 2018-04-02 16:45:35
tags: "React"
categories: "React"
---

# Context API

Context API总是很让人迷惑。这个API是官方的，但是官方又不希望开发者们使用这个API，说是这个API会在以后发生改变。现在就是那个改变的时刻。新的API已经被merge了。而且它看起来更加的“用户友好”了。尤其是你不得不使用redux、mobx的时候，可以选择新的Context API实现更加简单的状态管理。

新的API用起来非常的简单：**React.createContext()**，这样就创建了两个组件：
```javascript
import {context} from 'react';
const ThemeContext = createContext({
  background: 'yellow',
  color: 'white'
});
```
调用createContext方法会返回两个对象，一个是`Provider`，一个是`Consumer`。

那个Provider是一个特殊的组件。它可以用来给子树里的组件提供数据。一个例子：
```javascript
class Application extends React.Component {
  render() {
    <ThemeContext.Provider value={{background: 'black', color: 'white'}}>
      <Header />
      <Main />
      <Footer />
    </ThemeContext.Provider>
  }
}
```
上例展示了如何传递“theme” context的。当然这些值可以是动态的（比如，基于this.state）。

<!-- more -->

下一步就是使用Consumer。
```javascript
const Header = () => {
  <ThemeContext.Consumer>
    {(context) => {
      return (
        <p style={{background: context.background, color: context.color}}>
          Welcome!
        </p>
      );
    }}
  </ThemeContext.Consumer>
}
```
接下来，定义Header、Title组件。注意：

Title组件用到了Consumer组件，表示要消费Provider传递的数据。
Title组件是App的孙组件，但跳过了Header消费数据。

```javascript
    class Header extends React.Component {
  render () {
    return (
      <Title>Hello React Context API</Title>
    );
  }
}
 
class Title extends React.Component {
  render () {
    return (
      <ThemeContext.Consumer>
        {context => (
          <h1 style={{background: context.background, color: context.color}}>
            {this.props.children}
          </h1>
        )}
      </ThemeContext.Consumer>
    );
  }
}
```


如果在render Consumer的时候没有嵌套在一个Provider里面。那么就会使用createContext方法调用的时候设置的默认值。

**注意：**
Consumer必须可以访问到同一个Context组件。如果你要创建一个新的context，用的是同样的入参，那么这个新建的context的数据是不可访问的。因此，可以把Context当做一个组件，它可以创建一次，然后可以export，可以import。

这个新的语法用了function as child模式（有时也叫做render prop模式）。如果不是很熟悉这个模式，那么推荐你看一下这些文章。
新的API不再要求你声明`contextProps`了。
Context传递的数据和Context.Provider组件的value属性是一样的。对Provider数据的修改会引起所有的消费者`（consumer）`重绘。

# 新的声明周期方法

参考这个RFC。新的声明周期方法会被引入，而旧的会被废弃。

这一改变主要是为了强制推行最佳实践。你可以看看这篇文章来了解一下为什么这些生命周期方法会变得很诡异。这些最佳模式在React 16的异步绘制模式(Async Mode)下显得非常重要。

> 要被废弃的方法：
componentWillMount-->使用componentDidMount代替
componentWillUpdate-->使用componentDidUpdate代替
componentWillReceiveProps-->使用一个新的方法：static getDerivedStateFromProps来代替。

不过这些并不会立刻发生，他们可以用到React 16.4。在React 17里将被彻底移除。如果你开启了StrictMode或者AsyncMode，可以通过这样的方式来使用，但是会收到警告

UNSAFE_componentWillMount

UNSAFE_componentWillReceiveProps

UNSAFE_componentWillUpdate

## static getDerivedStateFromProps
当componentWillReceiveProps我们需要其他的方式根据props的变动更新state。社区决定引入一个新的static方法来处理这个问题。

什么是静态方法？一个静态方法就是存在于类内，而不是类的实例内的方法。静态方法访问不到this，并且在声明的时候有static关键字在前面修饰。

但是，问题来了。既然这个方法没有办法访问this，那么如何调用this.setState呢？答案就是，不调用。这个方法直接返回需要更新的state的数据，或者返回null，如果没有什么需要更新的话。

```javascript

static getDerivedStateFromProps(nextProps, prevState) {
  if(nextProps.currentRow === prevState.lastRow) {
    return null;
  }
 
  return {
    lastRow: nextProps.currentRow,
    isCrollingDown: nextProps.curentRow > prevState.lastRow
  }
}
```

调用这个方法和之前调用this.setState的效果是一样的。只会修改这些返回的值，如果是null的话则不修改state。state的其他值都会保留。

值得注意是
你需要定义初始state的值。无论是在constructor里，或者是类属性。否则会报警告。
这个方法getDerivedStateFromProps()会在第一次挂载和重绘的时候都会调用到，因此你基本不用在constructor里根据传入的props来setState。

如果定义了getDerivedStateFromProps后，又定义了componentWillReceiveProps。那么，只有前者会被调用，并且你会收到一个警告。

一般你会使用一个回调来保证某些代码实在state更新之后才被调用的。那么，请把这些代码都移到componentDidUpdate里。
如果你不喜欢使用static关键字，那么你可以这样：
```javascript
ComponentName.getDerivedStateFromProps = (nextProps, prevState) => {
  // Your code here
}
```

## static mode

严格模式是一个新的方式来确保你的代码是按照最佳实践开发的。它实际是一个在React.StrictMode下的组件。它可以用在你的组件树的任何一部分上。

```javascript
import {StrictMode} from 'react'
 
class Application extends React.Component {
  render() {
    return (
      <StrictMode>
        <Context.Provider value={{background: 'black', color: 'white'}}>
          <Header />
          <Main />
          <Footer />
        </Context.Provider>
      </StrictMode>
    );
  }
}
```

如果一个在StricMode子树里的组件使用了componentWillMount方法，那么你会看到一个报错消息。

## asyncMode
异步模式在React.unsafe_AsyncMode下。使用AsncMode也会打开StrictMode模式下的警告。