---
title: 吃透Vue项目开发实践(中)
tags: 'Vue'
categories: 'Vue'
date: 2020-05-09 10:03:02
---

[原文地址](https://juejin.im/post/5e15932ee51d4540f02fae27)
# 概览

![概览](https://user-gold-cdn.xitu.io/2019/12/30/16f5676f7940d72a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# 问题

> 本节内容主要围绕下列问题展开：

* 如何编写原生组件，以及组件编写的思考与原则？
* 组件怎么通信？有哪些方式？
* 如何使用vuex 以及它的应用场景和原理
* 如何使用过滤器，编写自己的过滤器
* 如何使用 Jest 测试你的代码？TDD 与 BDD 的比较

## 如何编写原生组件

### 组件编写原理

vue 编写组件有两种方式，一种是单文件组件，另外一种函数组件。根据组件引入和创建还可以分为动态组件和异步组件。

动态组件`keep-alive`使之缓存。异步组件原理和异步路由一样，使用`import()`实现异步加载也就是按需加载。

所谓 vue 单文件组件，就是我们最常见的这种形式：

```js
<template lang="pug">	
.demo  	h1 hello
</template>
<script>
export default {
  name: 'demo',
  data() {
    return {}
  }
}
</script><style lang="scss" scoped>
.demo {
  h1 {
    color: #f00;
  }
}
</style>
```
这里的template 也可以使用 `render` 函数来编写

```js
Vue.component('demo', {
    render: function (createElement) {
      return createElement(
        'h1',	  'hello', 
        // ...    
      )  
    }
})
```

我们可以发现`render`函数写模版让我们更有编程的感觉。
对模版也可以编程，在vue 里面我们可以很容易联想到，很多地方都有两种写法，一种是 `template` ， 一种是`js`。
比如：对于路由，我们既可以使用`:to=""`，又可以使用`$router.push`，这也许是 vue 用起来比较爽的原因。

#### 函数式组件是什么呢？

`functional`，这意味它**无状态 (没有响应式数据)**，也没有**实例** (没有 this 上下文)

* 单文件形式 2.5.0+

```html
<template functional></template>
```

* Render 函数形式

```js
Vue.component('my-component', {
  functional: true,
  render function (createElement, context) {
    return createElement('div')	
  }
}
```

#### 为什么要使用函数组件呢？

最重要的原因就是**函数组件开销低**，也就是对性能有好处，在不需要响应式和this的情况下，写成函数式组件算是一种优化方案。

组件写好了，需要将组件注册才能使用

### 组件注册的两种方式

组件注册分为两种，一种是全局注册，一种是局部注册

`局部注册`就是我们常用的 ```Vue.component('s-button', { /* ... */ })```，比较简单不详细论述

`全局注册`上节已经提到，在`new Vue` 之前在 `mian.js `注册，这里还提到一种自动全局注册的方法 `**require.text**`

```js
import Vue from 'vue'
import upperFirst from 'lodash/upperFirst'
import camelCase from 'lodash/camelCase'
const requireComponent = require.context(  './components',  
// 是否查询其子目录
  false,  /Base[A-Z]\w+\.(vue|js)$/
)
requireComponent.keys().forEach(fileName => {
    // 获取组件配置 
    const componentConfig = requireComponent(fileName)
    const componentName = upperFirst( 
         camelCase(      
           // 获取和目录深度无关的文件名
            fileName.split('/').pop().replace(/\.\w+$/, '')
          )
    )  
    // 全局注册组件
    Vue.component(
      componentName,
      componentConfig.default || componentConfig  
    )
})
```

基本原理和全局注册一样，就是将 `components` 中的组件文件名，`appButton `变成 `AppButton` 作为注册的组件名。
把原来需要手动复制的，变成之间使用 `keys` 方法批量注册。

### 实践开发一个 button 组件

现在，我们以写一个简单的原生 `button`组件为例，探讨一下组件开发的一些关键点。 写之前，我们需要抓住 4 个核心的要点：

* **用哪个标签作为组件主体， `button` 还是 `div` 标签**
* **如何根据属性控制 `button` 组件的颜色 `color` 、形状 `type` 、大小 `size`**
* **如何处理 `button` 组件的点击事件**
* **如何去扩展 `button` 组件的内容*

> 这些思考点在其他原生组件开发和高阶组件封装里面也需要考虑到

首先看第一个问题，大部分原生组件第一考虑的地方，就是主要标签用原生 `<button></button>`标签还是用 去模拟。

> 为什么不考虑 `<input>`呢，因为 `<button></button>` 元素比 `<input>` 元素更容易添加内部元素。你可以在元素内添加HTML内容（像 、 甚至 `<img>`），以及 `::after` 和 `::before` 伪元素来实现复杂的效果，而 `<input>` 只支持文本内容。

下面分析这两种写法的优劣

**使用原生 `button` 标签的优势：**

1.  更好的标签语义化
2.  原生标签自带的 `buff` ，一些自带的键盘事件行为等

> 为什么说更好的语义化呢？有人可能会说，可以使用 `role` 来增强 `div` 的语义化。确实可以，但是可能存在问题——有些爬虫并不会根据 `role` 来确定这个标签的含义。

> 另外一点，对开发者来说， `<button></button>` 比 阅读起来更好。

**使用 `div` 模拟的优势：**

1.  不需要关心 `button` 原生样式带来的一些干扰，少写一些覆盖原生 `css` 的代码，更干净纯粹。
2.  全部用 `div` ，不需要再去找原生标签、深入了解原生标签的一些兼容相关的诡异问题。
3.  可塑性更强，也就是拓展性和兼容性更好。这也是大多数组件都会选择使用 `div` 作为组件主体的原因。

> 貌似 div 除了语义不是很好以外，其他方面都还行，但是具体用哪一种其实都可以，只要代码写的健壮适配性强，基本都没啥大问题。

我们这里使用原生 `<button></button>`作为主要标签，使用 `s-xx`作为 `class`前缀

> 为什么需要使用前缀，因为在有些时候，比如使用第三方组件，多个组件之间的 class 可能会产生冲突，前缀用来充当组件 css 的一个命名空间，不同组件之间不会干扰。

```html
<template lang="pug">   
button.s-button(:class="xxx" :style="xxx" )
     slot
</template>
```

然后，我们看第二个问题：

**如何根据属性来控制 `button` 的样式** 其实这个很简单，基本原理就是：

1. 使用 `props` 获取父组件传过来的属性。
2. 根据相关属性，生成不同的 `class`，使用 `:class="{xxx: true, xxx: 's-button--' + size}"` 这种形式，在 `style` 里面对不同的 `s-button--xxx` 做出不同的处理。

```html
<template lang="pug">
  button.s-button(:class="" :style="" )    slot
</template>
<script>
export default {  
  name: 's-button'  
  data: return {}  
  props: {    
    theme: {},    
    size: {},    
    type: {}  
  }
}
</script>  
```

**如何使用事件以及如何扩展组件**

扩展组件的原理，主要就是使用 `props` 控制组件属性，模版中使用 `v-if/v-show` 增加组件功，比如增加内部 ICON，使用 `:style class`控制组件样式。

![type](https://user-gold-cdn.xitu.io/2020/1/13/16f9d9414f682625?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

还要注意的是我们还要控制原生 `type` 类型，原生默认是 `submit`，这里我们默认设置为 `button`

```html
<template lang="pug"> 
button.s-button(:class="" :style="" :type="nativeType")   
slot   
s-icon(v-if="icon && $slots.icon" name="loading")
</template>
<script>
export default {  
  name: 's-button'  
  data: return {}  
  props: {    
    nativeType: {      
      type: String,      
      default: 'button'    
    },    
    theme: {},    
    size: {},    
    type: {}  
  }
}
</script>
```

控制事件，直接使用 `@click=""` + `emit`

```html
<template lang="pug">  
button.s-button(@click="handleClick")
</template>
<script>
export default {  
  methods: {    
    handleClick (evt) {      
      this.$emit('click', evt)    
      }  
    }
}
</script>
```

#### 最后总结下：

> 一般就直接使用 `template`单文件编写组件，需要增强 js编写能力可以使用 `render()`
常规编写组件需要考虑：1. 使用什么标签 2. 如何控制各种属性的表现 3. 如何增强组件扩展性 4. 如何处理组件事件

> 对响应式 `this`要求不高，使用函数 `functional`组件优化性能。

> 基础组件通常全局注册，业务组件通常局部注册

> 使用 `keys()`遍历文件来实现自动批量全局注册

> 使用 `import()` 异步加载组件提升减少首次加载开销，使用 `keep-alive + is`缓存组件减少二次加载开销

## 如何使用 vuex 以及它的应用

### 由来以及原理

我们知道组件中通信有以下几种方式：

#### 1. 父组件通过 `props` 传递给子组件，不详细论述

#### 2. 子组件通过 `emit` 事件传递数据给父组件,父组件通过 `on` 监听，也就是一个典型的 **订阅-发布**模型

> `@` 为 `v-on:` 的简写

```html
<template lang="pug">
<!--子组件-->    
div.component-1
<template>
<script>
export default {  
  mounted() {    
    this.$emit('eventName', params)  
    }}
</script>
```

```html
<!-- 父组件-->
<template lang="pug">    
Component-1(@eventName="handleEventName")
<template>
<script>
export default {  
  methods: {    
    handleEventName (params) {      
      console.log(params)    
    }  
}
</script>
```

#### 3. 集中式通信事件，主要用于简单的非父子组件通信

原理很简单其实就是在 `emit` 与 `on` 的基础上加了一个事件中转站 "bus"。我觉得更像是现实生活中的集线器。

普遍的实现原理大概是这样的 "bus" 为 `vue` 的一个实例，实例里面可以调用 `emit`, `off`, `on` 这些方法。

```js
var eventHub = new Vue()
// 发布
eventHub.$emit('add', params)
// 订阅/响应
eventHub.$on('add', params)
// 销毁
eventHub.$off('add', params)
```

但是稍微复杂点的情况，使用这种方式就太繁锁了。还是使用 vuex 比较好。

从某种意义而言，我觉得 vuex 不仅仅是它的一种进化版。

1.  使用 `store` 作为 **状态管理** 的仓库，并且引入了 **状态** 这个概念
2.  它的模型完全不一样了， `bus` 模型感觉像一个电话中转站
![中转站](https://user-gold-cdn.xitu.io/2020/1/13/16f9ddf76fc5b0fa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

而 vuex 的模型更像是集中式的代码仓库。

> 与 `git` 类似，它不能直接修改代码，需要参与者提交 `commit`，提交完的 `commit`修改仓库，仓库更新，参与者 `fetch` 代码更新自己的代码。不同的是代码仓库需要合并，而 `vuex` 是直接覆盖之前的状态。

![state](https://user-gold-cdn.xitu.io/2020/1/10/16f8ede3d74ab245?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 管理 vuex

#### 原理

> "store"基本上就是一个容器，它包含着你的应用中大部分的状态 (`state`)。Vuex 和单纯的全局对象有以下两点不同

* **响应式** （改变属性后，触发组件中的状态改变，触发 `dom` ）
* **不能直接改变状态** （唯一途径是提交 `mutation` )

![store](https://user-gold-cdn.xitu.io/2020/1/13/16f9deb7be3845f5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**基本用法**：就是在 `state` 里面定义各种属性，页面或组件组件中，直接使用 `$store.state`或者 `$store.getters`来使用。如果想要改变状态 `state`呢，就 `commit` 一个 `mutation`

但是假如我想提交一连串动作呢？可以定义一个 `action`，然后使用 `$store.dispatch` 执行这个 `action`

使用 `action` 不仅能省略不少代码，而且关键是 `action` 中可以使用异步相关函数，还可以直接返回一个 `promise`

而为什么不直接到 `mutation`中写异步呢？ 因为 `mutation` 一定是个同步，它是唯一能改变 state 的，一旦提交了 `mutation`， `mutation` 必须给定一个明确结果。否则会阻塞状态的变化。

下面给出常用 vuex 的使用方式

#### 新建 Store

新建一个 `store`并将其他各个功能化分文件管理

```js
import Vue from 'vue'
import Vuex from 'vuex'
import state from './states'
import getters from './getters'
import mutations from './mutations'
import actions from './actions'
import user from './modules/user'
Vue.use(Vuex)
export default new Vuex.Store({    
  //在非生产环境下，使用严格模式    
  strict: process.env.NODE_ENV !== 'production',     
  state,    
  getters,    
  mutations,    
  actions,    
  modules: {        
    user    
  }
})
```

**操作状态两种方式**

1.  获取状态

```js
console.log(store.state.count)
```

2.  改变状态

```js
store.commit('xxx')
```

#### 管理状态 states

> 单一状态树, 这也意味着，每个应用将仅仅包含一个 store 实例。单一状态树让我们能够直接地定位任一特定的状态片段，在调试的过程中也能轻易地取得整个当前应用状态的快照。

```js
// states 文件
export default {    
  count: 0
}
```

计算属性中返回，每当 `state` 中属性变化的时候, 其他组件都会重新求取计算属性，并且触发更新相关联的 DOM

```js
const Counter = {	
  template: '<div>{{count}}<div>',	
  computed: {		
    count() {			
      return store.state.count		
    }	
  }
}
```

#### 管理取值器 getters

> `getters` 相当于 `store` 的计算属性。不需要每次都要在计算属性中过滤一下，也是一种代码复用。 我们在 `getters`文件中管理

```js
export default {    
  count: (state) => Math.floor(state.count)
}
```

#### 管理 mutations

> 更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 事件类型 (type) 和 一个 回调函数 (handler)。这个回调函数就是我们实际进行状态更改的地方

使用 types 大写用于调试，在 `mutationTypes` 文件中 `export const ROUTE_ADD = 'ROUTE_ADD'`

然后在 `mutations` 文件中管理

```js
import * as MutationTypes from './mutationTypes'
export default {    
  [MutationTypes.ADDONE]: function(state) {
    state.count = state.count + 1    
  },    
  //...
}
```

```js
this.$store.commit(MutationTypes.ADDONE)
```

#### 管理 actions

和 `mutations` 类似， `actions` 对应的是 `dispatch`，不同的是 `action`可以使用异步函数，有种更高一级的封装。

```js
// 简化
actions: {  
  increment ({ commit }) {    
    setTimeout(() => {        
      commit(MutationTypes.ADDONE)    }
    , 1000)  
  }
}
// 触发
store.dispatch('increment')
```

> 上述用法都可以使用 **载荷**的形式，引入也可以使用 `mapXxxx` 进行批量引入，这里不详细论述，有兴趣可以查看官网。

#### 分模块管理状态

状态太多太杂，分模块管理是一个良好的代码组织方式。

```js
import count from './modules/count'
export default new Vuex.Store({    
  modules: {        
    count    
  }
})
```

每一个模块都可以有独立的相关属性

```js
import * as ActionTypes from './../actionTypes'
export default {    
  state: {        
    count: 0    
  },    
  mutations: {        
    ADD_ONE: function(state) {             
      state.count = state.count + 1        
      }    
  },    
  actions: {        
    [ActionTypes.INIT_INTENT_ORDER]: function({ commit }) {            
      commit('ADD_ONE')        
    }    
  },    
  getters: {        
    pageBackToLoan: (state) => Math.floor(state.count)    
  }
}
```
![如图所示](https://user-gold-cdn.xitu.io/2020/1/13/16f9e11d627d9b4d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 应用场景

`vuex` 主要有几个应用场景，一个是用于 **组件通信**和 **状态共享**，一个是用于 **数据缓存**，还有就是用于 **减少请求**。这些场景归根节点都是对于 **缓存**和 **共享**来说的。

#### 组件通信

首先，状态统一管理在仓库，就实现了组件通信的可能性。

当一个组件通过 `commit` 提交 `mutation` 就改了 `state`，其他组件就可以通过 `store.state`获取最新的 `state`，这样一来就相当于 **更新的值通过 `store` 传递给了其他组件**，不仅实现了一对一的通信，还实现了一对多的通信。

#### 状态共享

我们经常使用的一个场景就是 **权限管理**。

写权限管理时候，首次进入页面就要将权限全部拿到，然后需要分发给各个页面使用，来控制各个路由、按钮的权限。

```js
/** * 
 * 判断用户有没有权限访问页面 
 */
function hasPermission(routeItem, menus) {  
  return menus.some(menuItem => {    
    return menuItem.name === routeItem.name  
  })
}
/** * 
 * 递归过滤异步路由表，返回符合用户角色权限的路由表 * 
 * @param {*} routes 路由表 * 
 * @param {*} menus 菜单信息 
 * */
function filterRoutes(routes, menus) {  
  return routes.filter(routeItem => {    
    if (routeItem.hidden) {      
      return true    
    } else if (hasPermission(routeItem, menus)) {      
      const menuItem = menus.find(item => item.name === routeItem.name)      
      if (menuItem && menuItem.children && menuItem.children.length > 0) {        
        routeItem.children = filterRoutes(routeItem.children, menuItem.children)        
        if (!routeItem.children.length) return false      
        }      
        return true    
        } else {      
          return false    
          }  
      })
  }
  const permission = {  
    state: {    
      routers: constantRouterMap,    
      addRouters: [],    
      roles: [],    
      user_name: '',    
      avatar_url: '',    
      onlyEditor: false,    
      menus: null,    
      personal: true,    
      teamList: []  
    },  
    mutations: {}
  }
  export default permission
```

而且权限还可以被更改，更改后的权限直接分发到其他页面组件中。这个场景要是不使用 `vuex` ，代码将会比较复杂。

#### 数据缓存

`store` 是一个仓库，它从创建开始就一直存在，只有页面 `Vuex.store` 实例被销毁，state 才会被清空。具体表现就是刷新页面。

这个数据缓存适用于：页面加载后缓存数据，刷新页面请求数据的场景。在一般 `Hybrid`中，一般不存在刷新页面这个按钮，所以使用 vuex 缓存数据可以应对大多数场景。

```js
export default {    
  state: {        
    // 缓存修改手机号需要的信息        
    changePhoneInfo: {            
      nameUser: '',            
      email: '',            
      phone: ''        
    },    
  }
}
```

如果需要持久化缓存，结合浏览器或 APP 缓存更佳。

```js
export default {    
  // 登陆成功后，vuex 写入token，并写入app缓存，存储持久化    
  [ActionTypes.LOGIN_SUCCESS]: function(store, token) {        
    store.commit(MutationTypes.SET_TOKEN, token)        
    setStorage('token', token)        
    router.replace({ name: 'Home', params: { source: 'login' } })    
  }
}
```

#### 减少请求(数据缓存的变种)

在写后台管理平台时候，经常会有 `list` 选型组件，里面数据从服务端拿的数据。如果我们把这个 `list` 数据存储起来，下次再次使用，直接从 `store` 里面拿，这样我们就不用再去请求数据了。相当于减少了一次请求。

## 如何使用过滤器，编写自己的过滤器

### 原理

假设我现在有个需求，需要将性别0、1、2，分别转换成男、女、不确定这三个汉字展示。页面中多处地方需要使用。

```html
<template lang="pug">  
.user-info    
.gender      
  label(for="性别") 性别      
  span {{gender}}
</template>
```

完成这个需求，我们知道有 **4** 种方式：

1.  使用 computed 方法
2.  使用 methods 方法
3.  使用 utils 工具类
4.  使用 filters

**应该选择哪种方式呢？**

我从下面三个方面来论述这个问题

**1. 可实现性**

*  使用 `computed` 实现成功，我们知道 `computed` 不是一个函数是无法传参的，这里有个技巧， `return` 一个函数接受传过来的参数

```js
    // ...  
    computed: {    
      convertIdToName() {      
        return function(value) {        
          const item = list.find(function(item) {          
              return item.id === value        
            })        
            return item.name      
          }    
        }  
    }
```

![如图所示](https://user-gold-cdn.xitu.io/2020/1/13/16f9eb7a20ecdcdf?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

*  使用 `methods` 实现成功，这里直接可以传参数，一种常规的写法。

> 注意 `methods`、 `computed`和 `data` 相互之前是不能同名的

```js
  // ...  
  methods: {    
    convertIdToName(value) {      
      const item = list.find(function(item) {        
        return item.id === value      
      })      
      return item.name    
    }  
  }
```

*  使用 `utils` 和 `methods` 差不多基本上也可以实现
*  使用 `filter` 也是实现的，有个可以和 `methods` 、 `computed` 同名哦

```js
    filters: {    
      console.log(this.render)    
      convertIdToName(value) {      
        const item = list.find(function(item) {        
          return item.id === value      
        })      
        return item.name    
      }  
    },
```

**总的来说他们全部都可以实现这个需求**

**2. 局限性**

* `computed` 、 `methods` 和 `data` 三者互不同名，他们没办法被其他组件使用，除非通过 `mixins`
* `filters` 与 `utils` 无法访问 `this` ，也就是于响应式绝缘。但是通过定义全局 `filters` ，可以其他地方使用，另外还可以直接加载第三方 `filter` 和 `utils`

**3. 总结比较**

`filters` 与 `utils` 归属一对，他们既是脱离了 `this`，获得了自由，又是被 `this` 弃之门外。相反 `methods` 与 `computed` 与 `this` 紧紧站在一起，但又是无法获得自由。

### 例子

#### 编写一个简单的千分位过滤器

```js
export const thousandBitSeparator = (value) => {  
  return 
  value && (value.toString().indexOf('.') !== -1 ? value.toString().replace(/(\d)(?=(\d{3})+\.)/g, function($0, $1) {      
    return $1 + ',';    
  }) : 
  value.toString().replace(/(\d)(?=(\d{3})+$)/g, function($0, $1) {      
    return $1 + ',';    
  })
  );
}
```

#### 使用 vue-filter 插件

两款插件

[vue-filter](https://www.npmjs.com/package/vue-filter) 使用 `use` 引入

[vue2-filters](https://www.npmjs.com/package/vue-filters) 使用 `mixins` 引入

> 有需要的话，我一般就用第二个了，大多数都是自己写一下小过滤器

自定义过滤器之后，直接全局自动注册，其他地方都可以使用

#### 注册全局过滤器

遍历过滤属性值，一次性全部注册

```js
for (const key in filters) {    
  Vue.filter(key, filters[key])
}
```

## 如何使用 Jest 测试你的代码

### 原理

我们思考一下测试 js 代码需要哪些东西

1.  浏览器运行环境
2.  断言库

如果是测试 vue 代码呢？ 那得再加一个 vue 测试容器

### Jest + Vue

#### 安装依赖

```json
{    
  "@vue/cli-plugin-unit-jest": "^4.0.5",    
  "@vue/test-utils": "1.0.0-beta.29",    
  "jest": "^24.9.0",    
  // ...
}
```

### 项目配置

```js
// For a detailed explanation regarding each configuration property, visit:// https://jestjs.io/docs/en/configuration.html
module.exports = {  
  preset: '@vue/cli-plugin-unit-jest',  
  automock: false, "/private/var/folders/10/bb2hb93j34999j9cqr587ts80000gn/T/jest_dx",  
  clearMocks: true,  
  // collectCoverageFrom: null,  
  coverageDirectory: 'tests/coverage'  
  //...
}
```

### 单元测试

#### 测试 utils 工具类

> 对我们之前写的一个性别名称转换工具进行测试

```js
import { convertIdToName } from './convertIdToName'
describe('测试convertIdToName方法', () => {  
  const list = [    
    { id: 0, name: '男' },    
    { id: 1, name: '女' },    
    { id: 2, name: '未知' }  
  ]  
  it('测试正常输入', () => {    
    const usage = list    
    usage.forEach((item) => {      
      expect(convertIdToName(item.id, list)).toBe(item.name)    
    })
  })  
  it('测试非正常输入', () => {    
    const usage = ['a', null, undefined, NaN]    
    usage.forEach((item) => {      
      expect(convertIdToName(item, list)).toBe('')    
    })  
  })
})
```

![如图所示](https://user-gold-cdn.xitu.io/2020/1/13/16f9eda87016bfe2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
这样一测试，发现原来我们之前写的工具有这么多漏洞 测试正常输入全部通过了，非正常输入失败了，根据测试用例改进我们的代码

```js
export const convertIdToName = (value, list) => {  
  if (value !== 0 && value !== 1 && value !== 2) return ''  
  const item = list.find(function(item) {    
    return item.id === value  
  }) 
  return item.name
}
```

现在测试都通过了呢

#### 测试 components 单文件组件

> 对我们最简单的 `hello world` 进行测试

```html
<template lang="pug">  
.hello    h1 {{ msg }}
</template>
<script>
export default {  
  props: {    
    msg: String  
  }
}
</script>
```

```js
import { shallowMount } from '@vue/test-utils'
import HelloWorld from '@/components/HelloWorld.vue'
describe('HelloWorld.vue', () => {  
  it('renders props.msg when passed', () => {    
    const msg = 'new message'    
    const wrapper = shallowMount(HelloWorld, {      
      propsData: { msg }    
    })    
    expect(wrapper.text()).toMatch(msg)  
  })
})
```

#### 测试 api 请求

异步测试有几种常见写法

* `async` 与 `await`
* `done()`

> 简单的异步测试，测试一个简单的登陆请求

```js
export const login = (data) => post('/user/login', data)
```

测试代码

```js
import { login } from '@/api/index'
describe('login api', () => {  
  const response = {    
    code: '1000',    
    data: {}  
  }  
  const errorResponse = {    
    code: '5000',    
    data: {},   
    message: '用户名或密码错误'  
  }  
  it('测试正常登陆', async () => {    
    const params = {      
      user: 'admin',      
      password: '123456'    
    }    
    expect(await login(params)).toEqual(response)  
  })  
  it('测试异常登陆', async () => {    
    const params = {      
      user: 'admin',     
      password: '123123'    
    }    
    expect(await login(params)).toEqual(errorResponse)  
  })
})
```
![如图所示](https://user-gold-cdn.xitu.io/2020/1/13/16f9efc734a64747?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 功能模块测试

组件， `api`，工具这些零零碎碎都测试了，而且这些都是比较通用、和业务关系不大的代码，它们改动较少，测试到这里其实已经足够了，已经达到了 `20%` 的测试工作量了很大一部分代码的目的。

> 为什么我说只有 20% 的工作量呢？因为这些都是不怎么变化的逻辑，是一劳永逸的事情。长远来说占用的工作量确实很少。

但是有些情况业务还是必须得测，也就是必须要功能模块集成测试。

经常得回归的业务，那种迭代对原有的系统比较大，避免改动之后使旧的代码各种新的问题。这种经常回归测试，采用 `BDD` + 集成测试，比不停改 `bug` 要轻松的多。

### 快照测试

像版本一样，每次测试之后生成一个版本，比较与上一个版本的区别。 这是一种粒度及其小的测试，可以测试到每一个符号。

#### 比如我用它来测试一个配置文件的变动

这是我们一个配置文件

```js
export const api = {  
  develop: 'http://xxxx:8080',  
  mock: 'http://xxxx',  
  feature: 'http://xxxx',  
  test: 'http://xxxx',  
  production: 'http://xxxx'
}
export default api[process.env.NODE_ENV || 'dev']
```

使用快照测试

```js
import { api } from './config'
describe('配置文件测试', () => {  
  it('测试配置文件是否变动', () => {    
    expect(api).toMatchSnapshot({      
      develop: 'http://xxxx:8080',     
       mock: 'http://xxxx',      
       feature: 'http://xxxx',      
       test: 'http://xxxx',      
       production: 'http://xxxx'    
      })  
  })
})
```

使用快照第一次测试后，通过测试，代码被写入快照
![快照测试1](https://user-gold-cdn.xitu.io/2020/1/13/16f9f1b5e3b2e64f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
![快照测试2](https://user-gold-cdn.xitu.io/2020/1/13/16f9f1d9415200e8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
改动配置再次测试，未通过测试 这时如果要改变配置怎么办呢？ 只需同时改一下用例就好了。快照将再次写入快照生成版本2，这样配置改动也有根据了
![快照测试3](https://user-gold-cdn.xitu.io/2020/1/13/16f9f1eeb7396fdc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


### TDD 与 BDD

最近讨论比较多的算是 **测试驱动开发**和 **行为驱动开发**，其实总得来说是 4 种

1.  不写测试。好处是省时间，坏处当然就 `bug` 多，代码质量低。
2.  先写测试和测试用例，再写代码，也就是测试驱动开发。这样好处是代码比较健全，考虑的因素比较多。固定模块健壮性高，bug少。
3.  先写代码，再通过模拟用户的行为写测试。好处是思路清晰，如果测试用例够全面，基本上线基本没太大问题，回归测试也很好做。
4.  写代码之前写测试和用例，写完代码之后再写用户行为测试。这种代码质量就太高了，当然缺点就是费时间。

> 那么你是哪一种？ 反正我比较佛系哈，有的不写测试，也有的写满测试。