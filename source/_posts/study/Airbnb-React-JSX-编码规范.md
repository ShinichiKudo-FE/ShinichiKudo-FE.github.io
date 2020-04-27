---
title: Airbnb React/JSX 编码规范
date: 2018-06-28 11:40:28
tags: "编码风格"
categories: "React"
---

# 前言

算是最合理的React/JSX编码规范之一了

此编码规范主要基于目前流行的JavaScript标准，尽管某些其他约定(如async/await，静态class属性)可能在不同的项目中被引入或者被禁用。目前的状态是任何stage-3之前的规范都不包括也不推荐使用。

## Basic Rules 基本规范

每个文件只写一个模块.
但是多个[无状态模块](https://reactjs.org/docs/components-and-props.html#stateless-functions)可以放在单个文件中. eslint: [react/no-multi-comp](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/no-multi-comp.md#ignorestateless).
推荐使用JSX语法.
不要使用 `React.createElement`，除非从一个非JSX的文件中初始化你的app.

## 创建模块

class vs React.createClass vs stateless

* 如果你的模块有内部状态或者是refs, 推荐使用 `class extends React.Component` 而不是 `React.createClass`. eslint:

```js
const Listing = React.createClass({
  // ...
  render() {
    return <div>{this.state.hello}</div>;
  }
});

// good
class Listing extends React.Component {
  // ...
  render() {
    return <div>{this.state.hello}</div>;
  }
}
```

* 如果你的模块没有状态或是没有引用refs， 推荐使用普通函数（非箭头函数）而不是类:

```js
// bad
class Listing extends React.Component {
  render() {
    return <div>{this.props.hello}</div>;
  }
}

// bad (relying on function name inference is discouraged)
const Listing = ({ hello }) => (
  <div>{hello}</div>
);

// good
function Listing({ hello }) {
  return <div>{hello}</div>;
}
```

<!-- more -->

## Mixins

[不要使用Mixins](https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html)

## Naming 命名

* 扩展名：React模块使用.jsx扩展名。
* 文件名：文件名使用帕斯卡命名。如，ReservationCard.jsx。
* 引用命名: React模块名使用帕斯卡命名，实例使用骆驼式命名. eslint: react/jsx-pascal-case

```js
// bad
import reservationCard from './ReservationCard';

// good
import ReservationCard from './ReservationCard';

// bad
const ReservationItem = <ReservationCard />;

// good
const reservationItem = <ReservationCard />;
```

* 模块命名: 模块使用当前文件名一样的名称. 比如 ReservationCard.jsx 应该包含名为 ReservationCard的模块. 但是，如果整个文件夹是一个模块，使用 index.js作为入口文件，然后直接使用 index.js 或者文件夹名作为模块的名称:

```js
// bad
import Footer from './Footer/Footer';

// bad
import Footer from './Footer/index';

// good
import Footer from './Footer';
```

* 高阶模块命名: 对于生成一个新的模块，其中的模块名 `displayName` 应该为高阶模块名和传入模块名的组合. 例如, 高阶模块 `withFoo()`, 当传入一个 Bar 模块的时候， 生成的模块名 `displayName` 应该为 `withFoo(Bar)`.

> 为什么？一个模块的 displayName 可能会在开发者工具或者错误信息中使用到，因此有一个能清楚的表达这层关系的值能帮助我们更好的理解模块发生了什么，更好的Debug.

```js
// bad
export default function withFoo(WrappedComponent) {
  return function WithFoo(props) {
    return <WrappedComponent {...props} foo />;
  }
}

// good
export default function withFoo(WrappedComponent) {
  function WithFoo(props) {
    return <WrappedComponent {...props} foo />;
  }

  const wrappedComponentName = WrappedComponent.displayName
    || WrappedComponent.name
    || 'Component';

  WithFoo.displayName = `withFoo(${wrappedComponentName})`;
  return WithFoo;
}
```

* 属性命名: 避免使用DOM相关的属性来用作其他的用途。

> 为什么？对于`style` 和 `className`这样的属性名，我们都会默认它们代表一些特殊的含义，如元素的样式，CSS class的名称。在你的应用中使用这些属性来表示其他的含义会使你的代码更难阅读，更难维护，并且可能会引起bug。

```js
// bad
<MyComponent style="fancy" />

// good
<MyComponent variant="fancy" />
```

## Declaration 声明模块

* 不要使用 displayName 来命名React模块，而是使用引用来命名模块， 如 class 名称.

```js
// bad
export default React.createClass({
  displayName: 'ReservationCard',
  // stuff goes here
});

// good
export default class ReservationCard extends React.Component {
}
```

## Alignment 代码对齐

遵循以下的JSX语法缩进/格式.

```js
// bad
<Foo superLongParam="bar"
     anotherSuperLongParam="baz" />

// good, 有多行属性的话, 新建一行关闭标签
<Foo
  superLongParam="bar"
  anotherSuperLongParam="baz"
/>

// 若能在一行中显示, 直接写成一行
<Foo bar="bar" />

// 子元素按照常规方式缩进
<Foo
  superLongParam="bar"
  anotherSuperLongParam="baz"
>
  <Quux />
</Foo>
```

## Quotes 单引号还是双引号

* 对于JSX属性值总是使用双引号("), 其他均使用单引号('). eslint: jsx-quotes

> 为什么? HTML属性也是用双引号, 因此JSX的属性也遵循此约定.

```js
// bad
<Foo bar='bar' />

// good
<Foo bar="bar" />

// bad
<Foo style={{ left: "20px" }} />

// good
<Foo style={{ left: '20px' }} />
```

## Spacing 空格

* 总是在自动关闭的标签前加一个空格，正常情况下也不需要换行.

```js
// bad
<Foo/>

// very bad
<Foo                 />

// bad
<Foo
 />

// good
<Foo />
```

* 不要在JSX {} 引用括号里两边加空格

```js
// bad
<Foo bar={ baz } />

// good
<Foo bar={baz} />
```

## Props 属性

* JSX属性名使用骆驼式风格camelCase.

```js
// bad
<Foo
  UserName="hello"
  phone_number={12345678}
/>

// good
<Foo
  userName="hello"
  phoneNumber={12345678}
/>
```

* 如果属性值为 true, 可以直接省略.

```js
// bad
<Foo
  hidden={true}
/>

// good
<Foo
  hidden
/>

// good
<Foo hidden />
```

* `<img>` 标签总是添加 alt 属性. 如果图片以presentation(感觉是以类似PPT方式显示?)方式显示，alt 可为空, 或者`<img>` 要包含role="presentation".

```js
// bad
<img src="hello.jpg" />

// good
<img src="hello.jpg" alt="Me waving hello" />

// good
<img src="hello.jpg" alt="" />

// good
<img src="hello.jpg" role="presentation" />
```

* 不要在 alt 值里使用如 "image", "photo", or "picture"包括图片含义这样的词， 中文也一样.

> 为什么? 屏幕助读器已经把 img 标签标注为图片了, 所以没有必要再在 alt 里说明了.

```js
// bad
<img src="hello.jpg" alt="Picture of me waving hello" />

// good
<img src="hello.jpg" alt="Me waving hello" />
```

* 使用有效正确的 aria role属性值 ARIA roles.

```js
// bad - not an ARIA role
<div role="datepicker" />

// bad - abstract ARIA role
<div role="range" />

// good
<div role="button" />
```

* 不要在标签上使用 accessKey 属性.

> 为什么? 屏幕助读器在键盘快捷键与键盘命令时造成的不统一性会导致阅读性更加复杂.

```js
// bad
<div accessKey="h" />

// good
<div />
```

* 避免使用数组的index来作为属性key的值，推荐使用唯一ID.

```js
// bad
{todos.map((todo, index) =>
  <Todo
    {...todo}
    key={index}
  />
)}

// good
{todos.map(todo => (
  <Todo
    {...todo}
    key={todo.id}
  />
))}
```

* 对于所有非必须的属性，总是手动去定义defaultProps属性.

> 为什么? propTypes 可以作为模块的文档说明, 并且声明 defaultProps 的话意味着阅读代码的人不需要去假设一些默认值。更重要的是, 显示的声明默认属性可以让你的模块跳过属性类型的检查.

```js
// bad
function SFC({ foo, bar, children }) {
  return <div>{foo}{bar}{children}</div>;
}
SFC.propTypes = {
  foo: PropTypes.number.isRequired,
  bar: PropTypes.string,
  children: PropTypes.node,
};

// good
function SFC({ foo, bar, children }) {
  return <div>{foo}{bar}{children}</div>;
}
SFC.propTypes = {
  foo: PropTypes.number.isRequired,
  bar: PropTypes.string,
  children: PropTypes.node,
};
SFC.defaultProps = {
  bar: '',
  children: null,
};
```

* 尽可能少地使用扩展运算符

> 为什么? 除非你很想传递一些不必要的属性

**例外情况:**

* 使用了变量提升的高阶组件

```js
function HOC(WrappedComponent) {
  return class Proxy extends React.Component {
    Proxy.propTypes = {
      text: PropTypes.string,
      isLoading: PropTypes.bool
    };

    render() {
      return <WrappedComponent {...this.props} />
    }
  }
}  
```

只有在清楚明白扩展对象时才使用扩展运算符。这非常有用尤其是在使用Mocha测试组件的时候。

```js
export default function Foo {
  const props = {
    text: '',
    isPublished: false
  }

  return (<div {...props} />);
}
```

特别提醒：尽可能地筛选出不必要的属性。同时，使用prop-types-exact来预防问题出现。

```js
 //good
render() {
  const { irrelevantProp, ...relevantProps  } = this.props;
  return <WrappedComponent {...relevantProps} />
}

//bad
render() {
  const { irrelevantProp, ...relevantProps  } = this.props;
  return <WrappedComponent {...this.props} />
}
```

## Refs

总是在Refs里使用回调函数.

```js
// bad
<Foo
  ref="myRef"
/>

// good
<Foo
  ref={(ref) => { this.myRef = ref; }}
/>
```

## Parentheses 括号

* 将多行的JSX标签写在()里

```js
// bad
render() {
  return <MyComponent className="long body" foo="bar">
           <MyChild />
         </MyComponent>;
}

// good
render() {
  return (
    <MyComponent className="long body" foo="bar">
      <MyChild />
    </MyComponent>
  );
}

// good, 单行可以不需要
render() {
  const body = <div>hello</div>;
  return <MyComponent>{body}</MyComponent>;
}
```

## Tags 标签

* 没有子元素标签总是自己闭合

```js
// bad
<foo className="stuff"></foo>

//good
<foo className="stuff"/>
```

* 如果模块有多行的属性， 关闭标签时新建一行.

```js
// bad
<Foo
  bar="bar"
  baz="baz" />

// good
<Foo
  bar="bar"
  baz="baz"
/>
```

## Methods函数

* 使用箭头函数来获取本地变量.

```js
function ItemList(props) {
  return (
    <ul>
      {props.items.map((item, index) => (
        <Item
          key={item.key}
          onClick={() => doSomethingWith(item.name, index)}
        />
      ))}
    </ul>
  );
}
```

* 当在 render() 里使用事件处理方法时，提前在构造函数里把 this 绑定上去.

> 为什么? 在每次 render 过程中， 再调用 bind 都会新建一个新的函数，浪费资源.

```js
// bad
class extends React.Component {
  onClickDiv() {
    // do stuff
  }

  render() {
    return <div onClick={this.onClickDiv.bind(this)} />;
  }
}

// good
class extends React.Component {
  constructor(props) {
    super(props);

    this.onClickDiv = this.onClickDiv.bind(this);
  }

  onClickDiv() {
    // do stuff
  }

  render() {
    return <div onClick={this.onClickDiv} />;
  }
}
```

* 在React模块中，不要给所谓的私有函数添加 _ 前缀，本质上它并不是私有的.

> 为什么？_ 下划线前缀在某些语言中通常被用来表示私有变量或者函数。但是不像其他的一些语言，在JS中没有原生支持所谓的私有变量，所有的变量函数都是共有的。尽管你的意图是使它私有化，在之前加上下划线并不会使这些变量私有化，并且所有的属性（包括有下划线前缀及没有前缀的）都应该被视为是共有的。了解更多详情请查看Issue [#1024](https://github.com/airbnb/javascript/issues/1024), 和 [#490](https://github.com/airbnb/javascript/issues/490) 。

```js
// bad
React.createClass({
  _onClickSubmit() {
    // do stuff
  },

  // other stuff
});

// good
class extends React.Component {
  onClickSubmit() {
    // do stuff
  }

  // other stuff
}
```

* 在 render 方法中总是确保 return 返回值.

```js
// bad
render() {
  (<div />);
}

// good
render() {
  return (<div />);
}
```

## Ordering React 模块生命周期

* class extends React.Component 的生命周期函数:

1.可选的 static 方法
2.constructor 构造函数
3.getChildContext 获取子元素内容
4.componentWillMount 模块渲染前
5.componentDidMount 模块渲染后
6.componentWillReceiveProps 模块将接受新的数据
7.shouldComponentUpdate 判断模块需不需要重新渲染
8.componentWillUpdate 上面的方法返回 true， 模块将重新渲染
9.componentDidUpdate 模块渲染结束
10.componentWillUnmount 模块将从DOM中清除, 做一些清理任务
11.点击回调或者事件处理器 如 onClickSubmit() 或 onChangeDescription()
12.render 里的 getter 方法 如 getSelectReason() 或 getFooterContent()
13.可选的 render 方法 如 renderNavigation() 或 renderProfilePicture()
14.render render() 方法

* 如何定义 `propTypes`, `defaultProps`, `contextTypes`, 等等其他属性...

```js
import React from 'react';
import PropTypes from 'prop-types';

const propTypes = {
  id: PropTypes.number.isRequired,
  url: PropTypes.string.isRequired,
  text: PropTypes.string,
};

const defaultProps = {
  text: 'Hello World',
};

class Link extends React.Component {
  static methodsAreOk() {
    return true;
  }

  render() {
    return <a href={this.props.url} data-id={this.props.id}>{this.props.text}</a>;
  }
}

Link.propTypes = propTypes;
Link.defaultProps = defaultProps;

export default Link;

```

* React.createClass 的生命周期函数，与使用class稍有不同:

1.displayName 设定模块名称
2.propTypes 设置属性的类型
3.contextTypes 设置上下文类型
4.childContextTypes 设置子元素上下文类型
5.mixins 添加一些mixins
6.statics
7.defaultProps 设置默认的属性值
8.getDefaultProps 获取默认属性值
9.getInitialState 或者初始状态
10.getChildContext
11.componentWillMount
12.componentDidMount
13.componentWillReceiveProps
14.shouldComponentUpdate
15.componentWillUpdate
16.componentDidUpdate
17.componentWillUnmount
18.clickHandlers or eventHandlers like onClickSubmit() or onChangeDescription()
19.getter methods for render like getSelectReason() or getFooterContent()
20.Optional render methods like renderNavigation() or renderProfilePicture()
21.render

## isMounted

> 为什么? isMounted 反人类设计模式:(), 在 ES6 classes 中无法使用， 官方将在未来的版本里删除此方法.

## 更多编码规范

[javascript](https://github.com/airbnb/javascript)
[vue](https://cn.vuejs.org/v2/style-guide)

[原文地址](https://github.com/JasonBoy/javascript/tree/master/react)