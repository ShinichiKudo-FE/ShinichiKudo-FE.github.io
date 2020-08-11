---
title: 重学React(一)
tags: 'React'
categories: 'React'
date: 2020-08-06 13:45:44
---

# 前言 

这几天手头里项目完成了一个阶段，下个阶段暂时有个星期的时间，正好有时间可以回顾下`react`。因为之前是选择react比vue要早，但是在之前公司里的技术栈是vue,没办法只有把react放到一旁，开始了`vue`的踩坑。

心里还是对react有感情的，这样说的我好像个渣男。。哈哈，确实挺喜欢这两个的框架。
既然是重学，自己就定位为react的初学者，要认真，充满激情，先从撸官网开始吧。
[react中文文档](https://react.docschina.org/)

## 新建一个react应用

我是通过[create-react-app](https://www.html.cn/create-react-app/docs/getting-started/)脚手架工具搭建的react初始界面

简单的过了一遍`creact-react-app`的文档，发现自己使用vue开发过项目之后，再返回来学习react知识，感觉轻松多了，以前很多概念理解不了，不知道怎么实现，虽然现在看的是一些基础知识，但是给了自己很大的信心去重学react

使用下面命令，就可以看的一个简单的react应用了
```js
npx create-react-app my-app
cd my-app
yarn start 
```

## 开发前的准备

* 1. 下载了一些vscode中关于react的一些插件，如`Reactjs code snippets`等
* 2. 安装`prettier，eslint`与vscode完美结合，减少一些格式上的错误
* 3. 从react文档中的核心概念看起，边看边跟着敲敲

## 文档中一些自己觉得常用的知识

### 核心概念部分

* 1. react是用jsx后缀名的文件来呈现代码的(ps:不是必须的),后面结合ts的话就是tsx后缀名的文件，总之jsx文件的概念就是`all in js`，react秉承的也是这种，vue则是数据和视图分离，
`templete`标签里编写html部分，`script`标签中编写js部分，`style`标签中编写css部分，这种模式是和我们最初写一个网页的结构相似的，便于我们理解。react提倡的jsx理由是这样说的**React 并没有采用将标记与逻辑进行分离到不同文件这种人为地分离方式，而是通过将二者共同存放在称之为“组件”的松散耦合单元之中，来实现关注点分离。**

一个简单的react示例：
```js
const name = 'Josh Perez';
const element = <h1>Hello, {name}</h1>;

ReactDOM.render(
  element,
  document.getElementById('root')
);
```
* 2. react 数据传递是通过`props`自上而下传递，是属于单向数据流，且`props`是只读的不可更改的。而vue是可以双向数据绑定

目前为止，在 React 中有两种流行的方式来共享组件之间的状态逻辑: [render props](https://react.docschina.org/docs/render-props.html) 和[高阶组件](https://react.docschina.org/docs/higher-order-components.html)

* 3. [react生命周期](https://projects.wojtekmaj.pl/react-lifecycle-methods-diagram/) ![react life](https://static01.imgkr.com/temp/3d03a0c13a2f4da7a4763a1dba50289e.png)
  

* 4. react事件处理的几种方法
```js
// 匿名箭头函数
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
// bind方法
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>

// 或者在方法中直接使用箭头函数，官网说这是实验性语法，不推荐使用，不过这个方法真的很简单明了啊
 handleClick = () => {
  console.log('this is:', this);
}

// 常见的方法 声明方法后在constructor中在声明一次，比较麻烦，
constructor(props) {
  super(props);
  this.state = {isToggleOn: true};

  // 为了在回调中使用 `this`，这个绑定是必不可少的
  this.handleClick = this.handleClick.bind(this);
}
handleClick() {
  this.setState(state => ({
    isToggleOn: !state.isToggleOn
  }));
}
```

* 5. 通过花括号包裹代码，你可以在 [JSX 中嵌入任何表达式](https://react.docschina.org/docs/introducing-jsx.html#embedding-expressions-in-jsx), 就和写js文件一样
* 6. 表单中的`受控组件`(如`<input>`、 `<textarea>` 和 `<select>`在 React 中，可变状态（mutable state）通常保存在组件的 state 属性中，并且只能通过使用 setState()来更新)与`非受控组件 `(在 React 中，`<input type="file" />` 始终是一个非受控组件，因为它的值只能由用户设置，而不能通过代码控制。) 
