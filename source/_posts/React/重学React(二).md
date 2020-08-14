---
title: 重学React(二)
tags: ''
categories: ''
date: 2020-08-11 17:45:34
---

# 前言

上一篇看了react的核心概念后，大概心里对react的五脏六腑有个形状了，接着看文档中的高级指引部分

## react文档中高级指引部分的重点

### 代码分割

在通过动态 import() 语法。

```js
import("./math").then(math => {
  console.log(math.add(16, 26));
});
```

**使用React.Lazy**

`React.lazy` 函数能让你像渲染常规组件一样处理动态引入（的组件）

```js
const OtherComponent = React.lazy(() => import('./OtherComponent'));
```

### context（常用）

内部的一个组件状态通信的一个方法

定义：**Context 提供了一个无需为每层组件手动添加 props，就能在组件树间进行数据传递的方法**。

`theme-context.js`
```js
// 确保传递给 createContext 的默认值数据结构是调用的组件（consumers）所能匹配的！
export const ThemeContext = React.createContext({
  theme: themes.dark,
  toggleTheme: () => {},
});
```
`theme-toggler-button.js`
```js
// 有两种显示方式
<MyContext.Provider value={/* 某个值 */}>

<MyContext.Consumer>
  {value => /* 基于 context 值进行渲染*/}
</MyContext.Consumer>
```

```js
import {ThemeContext} from './theme-context';

function ThemeTogglerButton() {
  // Theme Toggler 按钮不仅仅只获取 theme 值，它也从 context 中获取到一个 toggleTheme 函数
  return (
    <ThemeContext.Consumer>
      {({theme, toggleTheme}) => (
        <button          onClick={toggleTheme}
          style={{backgroundColor: theme.background}}>

          Toggle Theme
        </button>
      )}
    </ThemeContext.Consumer>
  );
}

export default ThemeTogglerButton;
```

`app.js`
```js
import {ThemeContext, themes} from './theme-context';
import ThemeTogglerButton from './theme-toggler-button';

class App extends React.Component {
  constructor(props) {
    super(props);

    this.toggleTheme = () => {
      this.setState(state => ({
        theme:
          state.theme === themes.dark
            ? themes.light
            : themes.dark,
      }));
    };

    // State 也包含了更新函数，因此它会被传递进 context provider。
    this.state = {
      theme: themes.light,
      toggleTheme: this.toggleTheme,
    };
  }

  render() {
    // 整个 state 都被传递进 provider
    return (
      <ThemeContext.Provider value={this.state}>
        <Content />
      </ThemeContext.Provider>
    );
  }
}

function Content() {
  return (
    <div>
      <ThemeTogglerButton />
    </div>
  );
}

ReactDOM.render(<App />, document.root);
```

## refs

**Ref 转发是一项将 ref 自动地通过组件传递到其一子组件的技巧**
类似vue中的 `refs`,获取真实dom的属性