---
title: 拥抱Vue3系列之jsx语法
tags: 'Vue'
categories: 'Vue'
date: 2020-07-07 13:48:26
---

[原文地址](https://juejin.im/post/5f01d7886fb9a07e6835601a?utm_source=gold_browser_extension#heading-6)

在过去的一年中，Vue 团队一直都在开发 Vue.js 的下一个主要版本，就在 6 月底，尤大更新同步了 Vue 3 及其周边生态的状态([Vue 3: mid 2020 status update](https://github.com/vuejs/rfcs/issues/183))。

```js
if (isTrue("I am planning to use Vue 3 for a new project")) {
  if (isTrue("I need IE11 support")) {
    await IE11CompatBuild() // July 2020
  }
  if (isTrue("RFCs are too dense, I need an easy-to-read guide")) {
    await migrationGuide() // July 2020
  }
  if (isTrue("I'd rather wait until it's really ready") {
      await finalRelease() // Targeting early August 2020
  })
  run(`npm init vite-app hello-vue3`)
  return
}
```

我们可以看到，如果一切顺利的话，预计在 8 月份，Vue 3 的正式版本就可以和我们见面了，目前距离发布正式版还有一定的差距，还要做一些兼容性的工作。同时还会提供对 IE11 的支持。

Vue 3 为了达到更快、更小、更易于维护、更贴近原生、对开发者更友好的目的，在很多方面进行了重构：

> 全面拥抱 TypeScript
> 重构 complier
> 重构 Virtual DOM
...

## 写在前面

这是该系列文章的第一篇，后续会持续更新，覆盖 `Vue 3`生态常用库。

JSX 是一个小众群体使用开发方式，第一篇以 JSX 为切入点，目标是让大多数开发 Vue 的同学也对 JSX 有一定的认知，在用 Vue 开发复杂应用时，也能有更加灵活的方式。
比如当开始写一个只能通过 `level prop` 动态生成标题 (heading) 的组件时，你可能很快想到这样实现：

```js
<script type="text/x-template" id="anchored-heading-template">
  <h1 v-if="level === 1">
    <slot></slot>
  </h1>
  <h2 v-else-if="level === 2">
    <slot></slot>
  </h2>
  <h3 v-else-if="level === 3">
    <slot></slot>
  </h3>
</script>
```

这里用模板并不是最好的选择，在每一个级别的标题中重复书写了 `<slot></slot>`，不够优雅。

如果尝试用 JSX 来写，代码就会变得简单很多。

```js
const App = {
  render() {
    const tag = `h${this.level}`
    return <tag>{this.$slots.default}</tag>
  }
}
```

看过 [Ant Design Vue](https://github.com/vueComponent/ant-design-vue) 源码 (下面简称为 `antdv`) 的同学应该知道， `antdv` 的底层是基于 JSX 来实现的，也是 Vue 生态中使用 JSX 的深度用户。

antd 为了尽快的兼容 Vue 3，和 Vue 官方展开合作，于是有了 [@ant-design-vue/babel-plugin-jsx](https://github.com/vueComponent/jsx)。

## Vue JSX 简介

对于使用 React 的开发者来说，JSX 再熟悉不过了，但是如果你是一个 Vue 的重度用户，可能对 JSX 不是特别熟悉，甚至听到有同学说没有 template 的 Vue 项目没有灵魂。

先来看下面一段代码：

```js
const el = <div>Vue 3</div>;
```

这段代码既不是 HTML 也不是字符串，被称之为 JSX，是 JavaScript 的扩展语法。JSX 可能会使人联想到模板语法，但是它具备 Javascript 的完全编程能力。

看到这里可能会有疑问，不少同学可能会以为 JSX 是 React 中特有的，其实不然。

大多数同学都知道，我们平常在 `.vue` 文件中开发的代码，实际上会被 vue-loader 处理，但可能少数同学去看过我们手把手写出的代码，会变编译成啥样。

有兴趣的同学可以戳这个地址来看下。[vue-template-explorer](https://vue-template-explorer.netlify.app/) (因为众所周知的原因，可能访问略慢)

```js
<div id="app">{{ msg }}</div>
```

```js
function render() {
  with(this) {
    return _c('div', {
      attrs: {
        "id": "app"
      }
    }, [_v(_s(msg))])
  }
}
```

观察上述代码我们发现，到运行阶段实际上都是 render 函数在执行。Vue 推荐在绝大多数情况下使用 template 来创建你的 HTML。然而在一些场景中，就需要使用 render 函数，它比 template 更加灵活。

写过 render 函数的同学可能深有体会，书写复杂的 render 函数异常痛苦，而且难以维护，非常容易被称之为 “祖传代码”。好在2.0 的官方提供了一个 [Babel 插件](https://link.zhihu.com/?target=https%3A//github.com/vuejs/babel-plugin-transform-vue-jsx)，可以将更接近于模板语法的 JSX 转译成 JavaScript。

使用过 React 的同学对于如何写 JSX 语法一定非常熟悉了，然而，Vue 2 中 的 JSX 写法和 React 还是有一些略微的区别。React 中所有传递的数据都挂在顶层。

```js
const App = <A className="x" style={style} onChange={onChange} />
```

Vue 2 中，仅仅属性就有三种：`组件属性 props，普通 html 属性attrs，DOM 属性 domProps`。想要更多了解如何在 Vue 2 中写 JSX 语法，可以看这篇，[在 Vue 中使用 JSX 的正确姿势](https://zhuanlan.zhihu.com/p/37920151)。

## Vue 3 中对 JSX 带来的改变

### 属性传递

Vue 3 中，属性这块的传递和 React 类似，意味这不需要再传递 props，attrs 这些属性。

```js
// before
{
  class: ['foo', 'bar'],
  style: { color: 'red' },
  attrs: { id: 'foo' },
  domProps: { innerHTML: '' },
  on: { click: foo },
  key: 'foo'
}

// after
{
  class: ['foo', 'bar'],
  style: { color: 'red' },
  id: 'foo',
  innerHTML: '',
  onClick: foo,
  key: 'foo'
}
```

### 指令改版

Vue 3 把大多数全局 API 和 内部 helper 移到了 ES 模块中导出(譬如 `v-model、transition、teleport`)，从而使得 Vue 3 在增加了很多新特性之后，基线的体积反而小了。

`v-model、v-show` 这些 API 全部通过模块导出的方式来引入

> 基线体积： 无法舍弃的代码体积

我们来看一段非常简单的代码 `<input v-model="x" />`，在 Vue 2 和 Vue 3 中的编译结果有何不同。

```js
// Vue 2 before
function render() {
  with(this) {
    return _c('input', {
      directives: [{
        name: "model",
        rawName: "v-model",
        value: (x),
        expression: "x"
      }],
      domProps: {
        "value": (x)
      },
      on: {
        "input": function ($event) {
          if ($event.target.composing) return;
          x = $event.target.value
        }
      }
    })
  }
}
```

```js
// Vue 3 after
import { vModelText as _vModelText, createVNode as _createVNode, withDirectives as _withDirectives, openBlock as _openBlock, createBlock as _createBlock } from "vue"

export function render(_ctx, _cache) {
  return _withDirectives((_openBlock(), _createBlock("input", {
    "onUpdate:modelValue": $event => (_ctx.x = $event)
  }, null, 8 /* PROPS */, ["onUpdate:modelValue"])), [
    [_vModelText, _ctx.x]
  ])
}
```

可以看到在 Vue 3 中，对各个 API 做了更加细致的拆分，理想状态下，用户可以在构建时利用摇树优化 (`tree-shaking`) 去掉框架中不需要的特性，只保留自己用到的特性。

模版编译器会生成适合做 `tree-shaking` 的代码，不需要使用者去关心如何去做，这部分的改动同样需要在 JSX 写法中实现。

模板编译器中增加了 `PatchFlag`，在 JSX 的编译过程同样也做了处理，性能会有提升，但是考虑到 JSX 的灵活性，做了一些兼容处理，该功能还在测试阶段。

## 从 Vue 2 到 Vue 3 的过渡

Vue 3 虽然引入了一部分破坏性的更新，但对于绝大多数 Vue 2 的 API 还是兼容的。那么同样的，我们也要尽可能让使用 JSX 的用户通过最小的成本升级到 Vue 3，这是一个核心的目标。
写这篇文章的时候，antdv 已经使用 [@ant-design-vue/babel-plugin-jsx](https://github.com/vueComponent/ant-design-vue) 重构了大约 70% 的功能，预计会在 Vue 3 正式版之前发布测试版，大概率会是东半球最快兼容 Vue 3 的企业级组件库。

### Vue 3 JSX 的 API 设计

* 函数式组件

```js
const App = () => <div>Vue 3 JSX</div>
```

* 普通组件

```js
const App = {
  render() {
    return <div>Vue 3.0</div>
  }
}
```

```js
const App = defineComponent(() => {
  const count = ref(0);

  const inc = () => {
    count.value++;
  };

  return () => (
    <div onClick={inc}>
      {count.value}
    </div>
  )
})
```

* Fragment

```js
const App = () => (
  <>
    <span>I'm</span>
    <span>Fragment</span>
  </>
)
```

Fragment 参考 React 的写法，尽可能写起来更加方便。

* Attributes/Props

```js
const App = () => <input type="email" />

const placeholderText = 'email'
const App = () => (
  <input
    type="email"
    placeholder={placeholderText}
  />
)
```

* 指令

> 建议在 JSX 中使用驼峰 (vModel)，但是 v-model 也能用

**v-show**

```js
const App = {
  data() {
    return { visible: true };
  },
  render() {
    return <input vShow={this.visible} />;
  },
};
```

**v-model**

> 修饰符：使用 (_) 代替 (.) (vModel_trim={this.test})

```js
export default {
 data: () => ({
   test: 'Hello World',
 }),
 render() {
   return (
     <>
       <input type="text" vModel_trim={this.test} />
       {this.test}
     </>
   )
 },
}
```

**自定义指令**

```js
const App = {
  directives: { antRef },
  setup() {
    return () => (
      <a
        vAntRef={(ref) => { this.ref = ref; }}
      />
    );
  },
}
```

* 插槽

关于指令、插槽最终的 API 还在讨论中，有想法的可以去留言。[Vue 3 JSX Design](https://github.com/vuejs/jsx/issues/141)

### Vue 2 的 JSX 写法如何快速迁移到 Vue 3

由于 antdv 的底层基本上都是基于 JSX 来写的，想要快速迁移到 Vue 3，就必须有一个比较好的插件来支持，这也是为什么会有这个插件的原因。当然在实现过程中也踩了很多坑。

目前用法和 Vue 2 的语法大多数是一致的，为了帮助更快迁移，在插件中做了针对旧 VNode 格式的兼容层，这里只能兼容一部分写法，以及部分语法的兼容会增加运行时的性能开销，所以我们希望能够将我们的经验分享给大家，让大家少走弯路！

```js
{
  "plugins": ["@ant-design-vue/babel-plugin-jsx", { "transformOn": true, "compatibleProps": true }]
}
```

* transformOn

针对 Vue 2 中 `on: { click: xx }` 写法的兼容，在运行时中会转为 `onClick: xxx`。

* compatibleProps

上文提到 Vue 3 对属性的传递做了变更，`props、attrs` 这些都不存在了，因此如果设置了这个属性为 true，在运行时也会被解构到第一层的属性中。

需要注意的一点，目前一旦开启这两个属性，在 `createVNode` 的第二个参数，都会包一个 `compatibleProps` 和 `transformOn` 方法，所以酌情开启这两个参数。对于使用 Vue 2 的 JSX 同学，如果没有使用到比较”不为人知“ 的 API的情况下，都可以快速得迁移。

那么 antdv 又是如何做迁移的呢？考虑到 antdv 是个组件库，都包一层 `compatibleProps` 势必不太优雅，因此没有选择开启这个两个开关。这里插一句，目前 antdv 的迁移还在进行中，相关的进度都在这个 issue 里面（[Vue 3 支持](https://github.com/vueComponent/ant-design-vue/issues/1913)），有兴趣的同学可以关注下，提一些 PR 过去。

对于 props 的迁移工作比较简单，只需要把原有分散在 `props、on、attrs` 中的值直接铺开即可。

```js
 const vcUploadProps = {
-  props: {
-    ...this.$props,
-   prefixCls,
-    beforeUpload: this.reBeforeUpload,
-  },
-  on: {
-    start: this.onStart,
-    error: this.onError,
-    progress: this.onProgress,
-    success: this.onSuccess,
-    reject: this.onReject,
- },
+  ...this.$props,
+  prefixCls,
+  beforeUpload: this.reBeforeUpload,
+  onStart: this.onStart,
+  onError: this.onError,
+  onProgress: this.onProgress,
+  onSuccess: this.onSuccess,
+  onReject: this.onReject,
+  ref: 'uploadRef',
+  attrs: this.$attrs,
+  ...this.$attrs,
};
```

但是关于 `inheritAttrs` 有个较为底层的变动，需要开发者根据实际情况去修改。[什么是inheritAttrs?](https://cn.vuejs.org/v2/api/index.html#inheritAttrs) 在 Vue 2 中，这个选项不影响 class 和 style 绑定，但是在 Vue 3 中会影响到。因此可能在属性的传递上，需要额外对这两个参数做处理。

在事件的处理上，我们建议在 props 中声明，这样对后续的开发更加易维护，可以很直观地从 props 看出我这个组件到底会传递哪些事件。值得一提的是，在 props 中声明的事件，也可以通过 emit 来触发。例如声明了 onClick 事件，仍然可以使用 emit('click')。

Vue 3 对 context 的 API 也做了改动，一般如果不是复杂的组件，不会涉及到这个 API。这部分的改动可以看原先 `Vue Compositon API` 的相关文档，[Dependency Injection](https://composition-api.vuejs.org/api.html#dependency-injection)，注意一点，在 setup 中取不到 this。
