---
title: 尤雨溪在Vue3.0 Beta直播里聊到了这些…
date: 2020-04-22 22:07:49
tags: "Vue3.x"
categories: "Vue"
---

[原文地址(https://juejin.im/post/5e9f6b3251882573a855cd52?utm_source=gold_browser_extension#heading-15)](https://juejin.im/post/5e9f6b3251882573a855cd52?utm_source=gold_browser_extension#heading-15)

# 前言

4月21日晚，Vue作者尤雨溪在哔哩哔哩直播分享了Vue.js 3.0 Beta最新进展。 以下是直播内容整理

## 全新文档RFCs

![全新文档RFCs](https://user-gold-cdn.xitu.io/2020/4/22/1719da838e76e8c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`Vue.js 3.0 Beta`发布后的工作重点是保证稳定性和推进各类库集成

所有的进度和文档都将在全新`RFCs`文档可以看到。

## 六大亮点

![六大亮点](https://user-gold-cdn.xitu.io/2020/4/22/1719daca80f3b7c2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> * Performance：性能更比Vue 2.0强。
> * Tree shaking support：可以将无用模块“剪辑”，仅打包需要的。
> * Composition API：组合API
> * Fragment, Teleport, Suspense：“碎片”，Teleport即Protal传送门，“悬念”
> * Better TypeScript support：更优秀的Ts支持
> * Custom Renderer API：暴露了自定义渲染API

### Performance

![Performance](https://user-gold-cdn.xitu.io/2020/4/21/1719d6747e723d03?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 1. 重写了虚拟Dom的实现（且保证了兼容性，脱离模版的渲染需求旺盛）。
* 2. 编译模板的优化。
* 3. 更高效的组件初始化。
* 4. update性能提高1.3~2倍。
* 5. SSR速度提高了2~3倍。

下面是各项性能对比

![性能对比](https://user-gold-cdn.xitu.io/2020/4/22/1719db04999e3890?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

#### 要点1：编译模板的优化

![编译模板的优化](https://user-gold-cdn.xitu.io/2020/4/22/1719e59cd0f2b668?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

假设要编译以下代码

```js
<div>
  <span/>
  <span>{{ msg }}</span>
</div>
```

```js
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", null, "static"),
    _createVNode("span", null, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}

// Check the console for the AST
```

注意看第二个`_createVNode`结尾的“1”：

Vue在运行时会生成number（大于0）值的`PatchFlag`，用作标记。

![标记](https://user-gold-cdn.xitu.io/2020/4/22/1719e69419edbc4d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

仅带有`PatchFlag`标记的节点会被真正追踪，且无论层级嵌套多深，它的动态节点都直接与`Block`根节点绑定，无需再去遍历静态节点

再看以下例子：

![例子2](https://user-gold-cdn.xitu.io/2020/4/22/1719e6d7d3efe7f1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

```js
<div>
  <span>static</span>
  <span :id="hello" class="bar">{{ msg }}   </span>
</div>
```

```js
import { createVNode as _createVNode, toDisplayString as _toDisplayString, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", null, "static"),
    _createVNode("span", {
      id: _ctx.hello,
      class: "bar"
    }, _toDisplayString(_ctx.msg), 9 /* TEXT, PROPS */, ["id"])
  ]))
}
```
<!-- more -->
`PatchFlag` 变成了`9 /* TEXT, PROPS */, ["id"]`
它会告知我们不光有TEXT变化，还有PROPS变化（id）
这样既跳出了`virtual dom`性能的瓶颈，又保留了可以手写`render`的灵活性。
等于是：既有react的灵活性，又有基于模板的性能保证。

#### 要点2: 事件监听缓存：cacheHandlers

![cacheHandlers](https://user-gold-cdn.xitu.io/2020/4/22/1719e775146d5a84?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

假设我们要绑定一个事件：

```html
<div>
  <span @click="onClick">
    {{msg}}
  </span>
</div>
```

关闭`cacheHandlers`后：

```js
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", { onClick: _ctx.onClick }, _toDisplayString(_ctx.msg), 9 /* TEXT, PROPS */, ["onClick"])
  ]))
}
```
`onClick`会被视为`PROPS`动态绑定，后续替换点击事件时需要进行更新。

开启`cacheHandlers`后：

```js
import { toDisplayString as _toDisplayString, createVNode as _createVNode, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return (_openBlock(), _createBlock("div", null, [
    _createVNode("span", {
      onClick: _cache[1] || (_cache[1] = $event => (_ctx.onClick($event)))
    }, _toDisplayString(_ctx.msg), 1 /* TEXT */)
  ]))
}
```

`cache[1]`，会自动生成并缓存一个内联函数，“神奇”的变为一个静态节点。 Ps：相当于`React中useCallback`自动化。

并且支持手写内联函数：

```html
<div>
  <span @click="()=>foo()">
    {{msg}}
  </span>
</div>
```

补充：`PatchFlags`枚举定义

而通过查询Ts枚举定义，我们可以看到分别定义了以下的追踪标记：
```js
export const enum PatchFlags {
  
  TEXT = 1,// 表示具有动态textContent的元素
  CLASS = 1 << 1,  // 表示有动态Class的元素
  STYLE = 1 << 2,  // 表示动态样式（静态如style="color: red"，也会提升至动态）
  PROPS = 1 << 3,  // 表示具有非类/样式动态道具的元素。
  FULL_PROPS = 1 << 4,  // 表示带有动态键的道具的元素，与上面三种相斥
  HYDRATE_EVENTS = 1 << 5,  // 表示带有事件监听器的元素
  STABLE_FRAGMENT = 1 << 6,   // 表示其子顺序不变的片段（没懂）。 
  KEYED_FRAGMENT = 1 << 7, // 表示带有键控或部分键控子元素的片段。
  UNKEYED_FRAGMENT = 1 << 8, // 表示带有无key绑定的片段
  NEED_PATCH = 1 << 9,   // 表示只需要非属性补丁的元素，例如ref或hooks
  DYNAMIC_SLOTS = 1 << 10,  // 表示具有动态插槽的元素
  // 特殊 FLAGS -------------------------------------------------------------
  HOISTED = -1,  // 特殊标志是负整数表示永远不会用作diff,只需检查 patchFlag === FLAG.
  BAIL = -2 // 一个特殊的标志，指代差异算法（没懂）
}
```

感兴趣的可以看源码：`packages/shared/src/patchFlags.ts`

### Tree shaking support

![Tree shaking support](https://user-gold-cdn.xitu.io/2020/4/22/1719db10e9e06382?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 可以将无用模块“剪辑”，仅打包需要的（比如`v-model`,`<transition>`，用不到就不会打包）。
* 一个简单“HelloWorld”大小仅为：13.5kb11.75kb，仅Composition API。
* 包含运行时完整功能：22.5kb,拥有更多的功能，却比Vue 2更迷你。

很多时候，我们并不需要 vue提供的所有功能，在 vue 2 并没有方式排除掉，但是 3.0 都可能做成了按需引入。

### Composition API

![Composition API](https://user-gold-cdn.xitu.io/2020/4/22/1719dc2469bd375c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

与React Hooks 类似的东西，实现方式不同。
 * 可与现有的 Options API一起使用
 * 灵活的逻辑组合与复用
 * vue 3的响应式模块可以和其他框架搭配使用

混入`(mixin)` 将不再作为推荐使用， `Composition API`可以实现更灵活且无副作用的复用代码。

感兴趣的可以查看：[composition-api.vuejs.org/#summary](https://composition-api.vuejs.org/#summary)

![omposition-api.vuejs.org](https://user-gold-cdn.xitu.io/2020/4/22/1719dc20717835c1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Composition API包含了六个主要API

[composition-api.vuejs.org/api.html#setup](https://composition-api.vuejs.org/api.html#setup)

Ps：其它的均为常见的工具函数，可先忽略不看。

### Fragment

![Fragment](https://user-gold-cdn.xitu.io/2020/4/22/1719dcd5c5f2b631?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Fragment翻译为：“碎片”

* 不再限于模板中的单个根节点
* render 函数也可以返回数组了，类似实现了 React.Fragments 的功能 。
* 'Just works'

#### <Teleport>

![Teleport](https://user-gold-cdn.xitu.io/2020/4/22/1719dd2d18fe0d55?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 以前称为`<Portal>`，译作传送门。
* 更多细节将由@`Linusborg` 分享

`<Teleport>`原先是对标 `React Portal`（增加多个新功能，更强）

但因为Chrome有个提案，会增加一个名为`Portal`的原生`element`，为避免命名冲突，改为`Teleport`

#### <Suspense>

![Suspense](https://user-gold-cdn.xitu.io/2020/4/22/1719ddbaf6d0f226?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Suspense翻译为：“悬念”

* 可在嵌套层级中等待嵌套的异步依赖项
* 支持async setup()
* 支持异步组件

虽然React 16引入了`Suspense`，但直至现在都不太能用。如何将其与异步数据结合，还没完整设计出来。
Vue 3 的`<Suspense>`更加轻量：
**仅5%应用能感知运行时的调度差异，综合考虑下，Vue3 的`<Suspense>` 没和React一样做运行调度处理**

### 更好的TypeScript支持

![TypeScript](https://user-gold-cdn.xitu.io/2020/4/22/1719de6ad581aa38?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


* Vue 3是用TypeScript编写的库，可以享受到自动的类型定义提示
* JavaScript和TypeScript中的API是相同的。事实上，代码也基本相同
* 支持TSX,class组件还会继续支持，但是需要引入`vue-class-component@next`，该模块目前还处在 alpha 阶段。

还有`Vue 3 + TypeScript` 插件正在开发，有类型检查，自动补全等功能。目前进展可喜。

![Vue 3 + TypeScript](https://user-gold-cdn.xitu.io/2020/4/22/1719deb6563506b7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Custom Renderer API：自定义渲染器API

![Custom Renderer API](https://user-gold-cdn.xitu.io/2020/4/22/1719df0e878dbeb3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 正在进行NativeScript Vue集成
* 用户可以尝试WebGL自定义渲染器，与普通Vue应用程序一起使用（Vugel）。

意味着以后可以通过 `vue`， `Dom` 编程的方式来进行 `webgl` 编程 。感兴趣可以看这里：[Getting started vugel](https://vugel.planning.nl/#application)

## 剩余工作

![剩余工作](https://user-gold-cdn.xitu.io/2020/4/22/1719df667b34919f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Docs & Migration Guides

![Docs & Migration Guides](https://user-gold-cdn.xitu.io/2020/4/22/1719df8c7c95f9b9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 新的文档编写交由`@NataliaTepluhina, @sdras, @bencodezen & @phanan` 负责
* `@sdras` 正在做自动升级迁移工具
* `@sodatea` 已经开始研究`CodeMods`

### Router

![Router](https://user-gold-cdn.xitu.io/2020/4/22/1719dfcab0e73db6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 下一代 Router：`vue-router@next`已在alpha阶段，感谢`@posva`

有部分的`API`变动，可到`RFC`上看。

### Vuex

![Vuex](https://user-gold-cdn.xitu.io/2020/4/22/1719dfed3afd961b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 下一代Vuex：，`vuex@next`（与Vue 3 compat相同的API），已在alpha阶段，感谢`@KiaKing`。
* 团队正在为下一次迭代试验Vuex API的简化

目前以兼容Vue 3为主，基本上没有API变动，莫慌。

### CLI

![CLI](https://user-gold-cdn.xitu.io/2020/4/22/1719e043616c018f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* CLI插件：`vue-cli-plugin-vue-next` by `@sodatea`
* （wip）CodeMods支持升级Vue 2应用

### 新工具：vite（法语 “快”）

![vite](https://user-gold-cdn.xitu.io/2020/4/22/1719e0c133c3367f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

地址：[github.com/vuejs/vite](https://github.com/vuejs/vite)

一个简易的http服务器，无需`webpack`编译打包，根据请求的Vue文件，直接发回渲染，且支持热更新（非常快）

### vue-test-utils

![vue-test-utils](https://user-gold-cdn.xitu.io/2020/4/22/1719e0d602d57dc6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### DevTools

![devtools](https://user-gold-cdn.xitu.io/2020/4/22/1719e0d0e479ec8b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### IDE Support (Vetur)

![IDE](https://user-gold-cdn.xitu.io/2020/4/22/1719e12c5bfa38e3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Nuxt

![Nuxt](https://user-gold-cdn.xitu.io/2020/4/22/1719e15306126103?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### Vue 2.x还有2.7版本

![Vue 2.x还有2.7版本](https://user-gold-cdn.xitu.io/2020/4/22/1719e16b0eaabdd2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

* 将有最后一个小版本（2.7）
* 从Vue 3向后移植兼容的改进(不损坏兼容性前提下)
* 加上在Vue 3中删除的功能的弃用警告
* LTS1 18个月。

**最后建议：Vue 3虽好，如果你的项目很稳定，且对新功能无过多的要求或者迁移成本过高，则不建议升级。**