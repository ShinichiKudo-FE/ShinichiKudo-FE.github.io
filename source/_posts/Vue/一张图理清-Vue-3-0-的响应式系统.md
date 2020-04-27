---
title: 一张图理清 Vue 3.0 的响应式系统
date: 2019-10-29 17:03:11
tags: "Vue3.x"
categories: "Vue"
---

转自

作者：Jrain
[链接](https://juejin.im/post/5d9da45af265da5b8072de5d)

![vue reactive](https://user-gold-cdn.xitu.io/2019/10/9/16dafca37b2e0534?imageslim)

## 一个基本的例子

Vue 3.0 的响应式系统是独立的模块，可以完全脱离 Vue 而使用，所以我们在 clone 了源码下来以后，可以直接在 `packages/reactivity` 模块下调试。

1.在项目根目录运行 `yarn dev reactivity`，然后进入 `packages/reactivity` 目录找到产出的 `dist/reactivity.global.js` 文件。
2.新建一个 `index.html`，写入如下代码：

```html
<script src="./dist/reactivity.global.js"></script>
<script>
const { reactive, effect } = VueObserver

const origin = {
  count: 0
}
const state = reactive(origin)

const fn = () => {
  const count = state.count
  console.log(`set count to ${count}`)
}
effect(fn)
</script>
```

3. 在浏览器打开该文件，于控制台执行 `state.count++`，便可看到输出 `set count to 1`。

在上述的例子中，我们使用 `reactive()` 函数把 `origin` 对象转化成了`Proxy` 对象 `state`；使用 `effect()` 函数把 `fn()` 作为响应式回调。当 `state.count` 发生变化时，便触发了 `fn()`。接下来我们将以这个例子结合上文的流程图，来讲解这套响应式系统是怎么运行的。

### 初始化阶段

> 在初始化阶段，主要做了两件事。

1.把 origin 对象转化成响应式的 Proxy 对象 state。
2.把函数 fn() 作为一个响应式的 effect 函数。


大家都知道，Vue 3.0 使用了 `Proxy` 来代替之前的 `Object.defineProperty()`，改写了对象的 `getter/setter`，完成依赖收集和响应触发。但是在这一阶段中，我们暂时先不管它是如何改写对象的 `getter/setter` 的，这个在后续的”**依赖收集阶段**“会详细说明。为了简单起见，我们可以把这部分的内容浓缩成一个只有两行代码的 `reactive()` 函数：


```js
    export default reactive (target){
        const observed = new Proxy(target,handler);
        return observed;
    }
```

当一个普通的函数 `fn()` 被 `effect()` 包裹之后，就会变成一个响应式的 effect 函数，而 `fn()` 也会被立即执行一次。

**由于在 fn() 里面有引用到 Proxy 对象的属性，所以这一步会触发对象的 getter，从而启动依赖收集**。

除此之外，这个 effect 函数也会被压入一个名为”`activeReactiveEffectStack`“（此处为 `effectStack`）的栈中，供后续依赖收集的时候使用。

```js
export function effect (fn) {
  // 构造一个 effect
  const effect = function effect(...args) {
    return run(effect, fn, args)
  }
  // 立即执行一次
  effect()
  return effect
}

export function run(effect, fn, args) {
  if (effectStack.indexOf(effect) === -1) {
    try {
      // 往池子里放入当前 effect
      effectStack.push(effect)
      // 立即执行一遍 fn()
      // fn() 执行过程会完成依赖收集，会用到 effect
      return fn(...args)
    } finally {
      // 完成依赖收集后从池子中扔掉这个 effect
      effectStack.pop()
    }
  }
}
```

至此，初始化阶段已经完成。接下来就是整个系统最关键的一步——依赖收集阶段。
<!-- more -->
### 依赖收集阶段

这个阶段的触发时机，就是在 `effect` 被立即执行，其内部的 `fn()` 触发了 `Proxy` 对象的 `getter` 的时候。简单来说，只要执行到类似 `state.count` 的语句，就会触发 `state 的 getter`。

依赖收集阶段最重要的目的，就是建立一份”依赖收集表“，也就是图示的”`targetMap`"。`targetMap` 是一个 `WeakMap`，其 key 值是~~当前的 `Prox`y 对象 `state`~~代理前的对象origin，而 value 则是该对象所对应的 `depsMap`。

`depsMap` 是一个 Map，key 值为触发 `getter` 时的属性值（此处为 count），而 value 则是触发过该属性值所对应的各个 effect。

还是有点绕？那么我们再举个例子。假设有个 Proxy 对象和 effect 如下：

```js
const state = reactive({
  count: 0,
  age: 18
})

const effect1 = effect(() => {
  console.log('effect1: ' + state.count)
})

const effect2 = effect(() => {
  console.log('effect2: ' + state.age)
})

const effect3 = effect(() => {
  console.log('effect3: ' + state.count, state.age)
})
```

这样，`{ target -> key -> dep }` 的对应关系就建立起来了，依赖收集也就完成了

```js
export function track (target, operationType, key) {
  const effect = effectStack[effectStack.length - 1]
  if (effect) {
    let depsMap = targetMap.get(target)
    if (depsMap === void 0) {
      targetMap.set(target, (depsMap = new Map()))
    }

    let dep = depsMap.get(key)
    if (dep === void 0) {
      depsMap.set(key, (dep = new Set()))
    }

    if (!dep.has(effect)) {
      dep.add(effect)
    }
  }
}

```
弄明白依赖收集表 targetMap 是非常重要的，因为这是整个响应式系统核心中的核心。

### 响应阶段

回顾上一章节的例子，我们得到了一个 `{ count: 0, age: 18 }` 的 Proxy，并构造了三个 effect。在控制台上看看效果：

![effect](https://user-gold-cdn.xitu.io/2019/10/9/16dafca37dadf75d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

当修改对象的某个属性值的时候，会触发对应的 `setter`。

setter 里面的 `trigger()` 函数会从依赖收集表里找到当前属性对应的各个 `dep`，然后把它们推入到 `effects` 和 `computedEffects`（计算属性） 队列中，最后通过 `scheduleRun()` 挨个执行里面的 effect。

由于已经建立了依赖收集表，所以要找到属性所对应的 dep 也就轻而易举了

```js
export function trigger (target, operationType, key) {
  // 取得对应的 depsMap
  const depsMap = targetMap.get(target)
  if (depsMap === void 0) {
    return
  }
  // 取得对应的各个 dep
  const effects = new Set()
  if (key !== void 0) {
    const dep = depsMap.get(key)
    dep && dep.forEach(effect => {
      effects.add(effect)
    })
  }
  // 简化版 scheduleRun，挨个执行 effect
  effects.forEach(effect => {
    effect()
  })
}
```

至此，响应式阶段完成。