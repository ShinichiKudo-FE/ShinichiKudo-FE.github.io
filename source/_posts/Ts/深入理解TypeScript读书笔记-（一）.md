---
title: <深入理解TypeScript读书笔记>（一）
date: 2020-03-25 21:07:50
tags: "Ts"
categories: "Js"
---

# 书本地址

[https://jkchao.github.io/typescript-book-chinese/#why](https://jkchao.github.io/typescript-book-chinese/#why)

## 什么是TypeScript,为什么要使用TypeScript?

根据TypeScript官网中的介绍，TypeScript - JavaScript the scales (属于Js的超集)，Ts发展至今已经是很多大型项目的标配，它提供的静态类型检查，大大的提高了代码的可维护性及可读性；同时提供最新和不断发展的Javascript特性，让我们能建立更加强壮的组件。

## 从tsconfig.json配置看ts整体

### 创建tsconfig.json文件，配置信息

通过compilerOption来定制你的编译选项

```json
{
  "compilerOptions": {

    /* 基本选项 */
    "target": "es5",                       // 指定 ECMAScript 目标版本: 'ES3' (default), 'ES5', 'ES2015', 'ES2016', 'ES2017', or 'ESNEXT'
    "module": "commonjs",                  // 指定使用模块: 'commonjs', 'amd', 'system', 'umd' or 'es2015'
    "lib": [],                             // 指定要包含在编译中的库文件
    "allowJs": true,                       // 允许编译 javascript 文件
    "checkJs": true,                       // 报告 javascript 文件中的错误
    "jsx": "preserve",                     // 指定 jsx 代码的生成: 'preserve', 'react-native', or 'react'
    "declaration": true,                   // 生成相应的 '.d.ts' 文件
    "sourceMap": true,                     // 生成相应的 '.map' 文件
    "outFile": "./",                       // 将输出文件合并为一个文件
    "outDir": "./",                        // 指定输出目录
    "rootDir": "./",                       // 用来控制输出目录结构 --outDir.
    "removeComments": true,                // 删除编译后的所有的注释
    "noEmit": true,                        // 不生成输出文件
    "importHelpers": true,                 // 从 tslib 导入辅助工具函数
    "isolatedModules": true,               // 将每个文件做为单独的模块 （与 'ts.transpileModule' 类似）.

    /* 严格的类型检查选项 */
    "strict": true,                        // 启用所有严格类型检查选项
    "noImplicitAny": true,                 // 在表达式和声明上有隐含的 any类型时报错
    "strictNullChecks": true,              // 启用严格的 null 检查
    "noImplicitThis": true,                // 当 this 表达式值为 any 类型的时候，生成一个错误
    "alwaysStrict": true,                  // 以严格模式检查每个模块，并在每个文件里加入 'use strict'

    /* 额外的检查 */
    "noUnusedLocals": true,                // 有未使用的变量时，抛出错误
    "noUnusedParameters": true,            // 有未使用的参数时，抛出错误
    "noImplicitReturns": true,             // 并不是所有函数里的代码都有返回值时，抛出错误
    "noFallthroughCasesInSwitch": true,    // 报告 switch 语句的 fallthrough 错误。（即，不允许 switch 的 case 语句贯穿）

    /* 模块解析选项 */
    "moduleResolution": "node",            // 选择模块解析策略： 'node' (Node.js) or 'classic' (TypeScript pre-1.6)
    "baseUrl": "./",                       // 用于解析非相对模块名称的基目录
    "paths": {},                           // 模块名到基于 baseUrl 的路径映射的列表
    "rootDirs": [],                        // 根文件夹列表，其组合内容表示项目运行时的结构内容
    "typeRoots": [],                       // 包含类型声明的文件列表
    "types": [],                           // 需要包含的类型声明文件名列表
    "allowSyntheticDefaultImports": true,  // 允许从没有设置默认导出的模块中默认导入。

    /* Source Map Options */
    "sourceRoot": "./",                    // 指定调试器应该找到 TypeScript 文件而不是源文件的位置
    "mapRoot": "./",                       // 指定调试器应该找到映射文件而不是生成文件的位置
    "inlineSourceMap": true,               // 生成单个 soucemaps 文件，而不是将 sourcemaps 生成不同的文件
    "inlineSources": true,                 // 将代码与 sourcemaps 生成到一个文件中，要求同时设置了 --inlineSourceMap 或 --sourceMap 属性

    /* 其他选项 */
    "experimentalDecorators": true,        // 启用装饰器
    "emitDecoratorMetadata": true          // 为装饰器提供元数据的支持
  }
}
```

### Ts编译生成Js文件

在目录下运行 `tsc`命令，会在当前目录或者父级目录下找`tsconfig.json`文件
运行 `tsc -p ./path-to-project-directory` 。当然，这个路径可以是绝对路径，也可以是相对于当前目录的相对路径。
使用 `tsc -w` 来启用 TypeScript 编译器的观测模式，在检测到文件改动之后，它将重新编译。

### 显示的指定编译文件

```js
{
  "files": [
    "./some/file.ts"
  ]
}
```

## 声明空间

### 类型声明空间（class interface type）

```js
class Foo{}
interface Bar{}
type Bas{}
```
**注意：**尽管你定义了 interface Bar，却并不能够把它作为一个变量来使用，因为它没有定义在变量声明空间中。
### 变量声明空间
```js
class Foo {}
const someOne = Foo
const someOneVar = 123
```
**注意** 一些用 var 声明的变量，也只能在变量声明空间使用，不能用作类型注解。

<!-- more -->

## 模块

### 全局模块

类似js的中全局对象，是一种不安全的方式，可使用文件模块代替

```js
const Foo = 123
const Bar = Foo
```

### 文件模块

文件模块也可以叫做外部模块，通过import,export导入导出，从而才能使用

```js
//foo.js
export const foo = 123
//bar.js
import {foo} from './foo.js'
const bar = foo
```

#### 文件模块详情

##### ES模块语法

导出
```js
//使用 export 关键字导出一个变量或类型
export const someVar = 123;
export type someType = {
  foo: string;
};

// export 的写法除了上面这种，还有另外一种：
const someVar = 123;
type someType = {
  type: string;
};

export { someVar, someType };

// 可以用重命名变量的方式导出：
const someVar = 123;
export { someVar as aDifferentName };
```
导入
```js
//使用 import 关键字导入一个变量或者是一个类型：
import { someVar, someType } from './foo';

//通过重命名的方式导入变量或者类型：
import { someVar as aDifferentName } from './foo'

//可以使用整体加载，即用星号（*）指定一个对象，所有输出值都加载在这个对象上面：
// 你可以使用 `foo.someVar` 和 `foo.someType` 以及其他任何从 `foo` 导出的变量或者类型
import * as foo from './foo';

//只导入模块：
import 'core-js'; // 一个普通的 polyfill 库

//从其他模块导入后整体导出：
export * from './foo';

//从其他模块导入后，部分导出：
export { someVar } from './foo';

//通过重命名，部分导出从另一个模块导入的项目：
export { someVar as aDifferentName } from './foo';

//默认导入／导出,使用 export default
// some var
export default (someVar = 123);

//导入使用 import someName from 'someModule' 语法（你可以根据需要为导入命名）：
import someLocalNameForThisFile from './foo';

```

##### 模块路径

这里存在两种截然不同的模块：

> 相对模块路径（路径以 . 开头，例如：./someFile 或者 ../../someFolder/someFile 等）；
> 其他动态查找模块（如：core-js，typestyle，react 或者甚至是 react/core 等）。

##### 什么是 place

这个点指的应该是一个模块查找的过程

##### 重写类型的动态查找

在项目里，可以通过`declare module 'somePath'` 声明一个全局模块的方式，来解决查找模块路径的问题。

##### import/require 仅仅是导入类型

它实际上只做了两件事：

* 导入 foo 模块的所有类型信息；
* 确定 foo 模块运行时的依赖关系。

## 命名空间

在 JavaScript 可以使用匿名函数创建一个命名空间

```js
(function(something) {
  something.foo = 123;
})(something || (something = {}));
```
TypeScript 提供了 namespace 关键字来描述这种分组
```js
namespace Utility {
  export function log(msg) {
    console.log(msg);
  }
  export function error(msg) {
    console.log(msg);
  }
}

// usage
Utility.log('Call me');
Utility.error('maybe');
```

## 动态导入表达式

动态导入表达式是 ECMAScript 的一个新功能，它允许你在程序的任意位置异步加载一个模块  