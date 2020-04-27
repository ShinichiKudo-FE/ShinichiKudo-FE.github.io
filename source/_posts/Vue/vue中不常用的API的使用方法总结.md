---
title: vue中不常用的API的使用方法总结
date: 2019-06-13 13:46:23
tags: "Vue"
categories: "Vue"
---

## errorHandler

官网介绍及使用：

指定组件的渲染和观察期间未捕获错误的处理函数。这个处理函数被调用时，可获取错误信息和 Vue 实例

```js
Vue.config.errorHandler = function (err, vm, info) {
  //处理错误信息, 进行错误上报
  //err错误对象
  //vm Vue实例
  //`info` 是 Vue 特定的错误信息，比如错误所在的生命周期钩子
  //只在 2.2.0+ 可用
}
```

## Vue.observable( object )

这个相当于一个简单的store管理，在不用vuex的情况下，不同组件之间通信可以用`Vue.observeable`

官网介绍：

让一个对象可响应。Vue 内部会用它来处理 data 函数返回的对象。

返回的对象可以直接用于渲染函数和计算属性内，并且会在发生改变时触发相应的更新。也可以作为最小化的跨组件状态存储器，用于简单的场景：

使用：

在store.js中

```js
impor vue from 'vue';
export const store = vue.observable({count:0});
export const mutation = {
    setCount(count){
        store.count = count;
    }
}
```

在组件中使用

```js
<template>
  <div class="hello">
    <p @click="setCount(testCount + 1)">+</p>
    <p @click="setCount(testCount - 1)">-</p>
    <test />
    <p>{{testCount}}</p>
  </div>
</template>

<script>
import test from './test'
import { store,  mutation} from '@/store'
export default {
  name: 'HelloWorld',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  components: {
    test
  },
  methods: {
    setCount: mutation.setCount
  },
  computed: {
    testCount(){
      return store.count
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>

```
<!-- more -->

test 组件

```js
<template>
  <div>test{{testCount}}</div>
</template>
<script>
import { store } from '@/store';
export default {
  computed: {
    testCount(){
      return store.count
    }
  }
}
</script>
<style scoped>

</style>
```

# errorCaptured(2.5.0新增的生命周期钩子)

官网介绍：

当捕获一个来自子孙组件的错误时被调用。此钩子会收到三个参数：错误对象、发生错误的组件实例以及一个包含错误来源信息的字符串。此钩子可以返回 false 以阻止该错误继续向上传播。

与errorhandler的不同之处：

errorCaptured 和 errorHandler 的触发时机都是相同的，不同的是 errorCaptured 发生在前，且如果某个组件的 errorCaptured 方法返回了 false，那么这个异常信息不会再向上冒泡也不会再调用 errorHandler 方法

# parent

官网介绍：

指定已创建的实例之父实例，在两者之间建立父子关系。子实例可以用 this.$parent 访问父实例，子实例被推入父实例的 $children 数组中。

注意:节制地使用 $parent 和 $children - 它们的主要目的是作为访问组件的应急方法。更推荐用 props 和 events 实现父子组件通信

使用:

```js
    this.$parent.xxx() //直接在子组件调用父组件中方法
```

# mixin

官网介绍：

mixins 选项接受一个混入对象的数组。这些混入实例对象可以像正常的实例对象一样包含选项，他们将在 Vue.extend() 里最终选择使用相同的选项合并逻辑合并。举例：如果你的混入包含一个钩子而创建组件本身也有一个，两个函数将被调用。

mixins就是定义一部分公共的方法或者计算属性,然后混入到各个组件中使用,方便管理与统一修改

1、在assets文件夹下创建一个js文件

```js
// 创建一个需要混入的对象 
export const mixinTest1 = {
    created() {
        this.hello();
    },
    methods: {
        hello() {
            console.log('mixinTest1');
        }
    }
};
```

2、在组件中使用刚刚创建的混入

```js
import {mixinTest1} from './../assets/js/mixin';
export default {
    mixins:[mixinTest1],
    name: 'hello',
    data () {
        return {
            msg: 'Welcome to Your Vue.js App'
        }
    }
}
```

3、如果组件中定义的方法与混入对象中的方法/属性一样,组件中的优先级大于混入对象中的(方法会调用多次)

4、混入对象中可以定义抽象方法,使用混入的组件必须重写该方法

```js
...
methods: {
    handlePlaylist() {
        throw new Error('component must implement handlePlaylist method')
    }
}
...
```

# provide / inject

官网介绍：

这对选项需要一起使用，以允许一个祖先组件向其所有子孙后代注入一个依赖，不论组件层次有多深，并在起上下游关系成立的时间里始终生效。如果你熟悉 React，这与 React 的上下文特性很相似。

provide 选项应该是一个对象或返回一个对象的函数。该对象包含可注入其子孙的属性。在该对象中你可以使用 ES2015 Symbols 作为 key，但是只在原生支持 Symbol 和 Reflect.ownKeys 的环境下可工作。

inject 选项应该是：

一个字符串数组，或
一个对象，对象的 key 是本地的绑定名，value 是：
在可用的注入内容中搜索用的 key (字符串或 Symbol)，或
一个对象，该对象的：
from 属性是在可用的注入内容中搜索用的 key (字符串或 Symbol)
default 属性是降级情况下使用的 value

* 注意：provide 和 inject 主要为高阶插件/组件库提供用例。并不推荐直接用于应用程序代码中。

我之前使用到`provide / inject`是**模拟页面数据回到初始状态**，相当于刷新当前页面，但是直接刷新当前页面会有几秒的空白期，体验不好，所以用到了`provide / inject`

使用方法：

1.修改App.vue代码如下图所示

```js
<template>
    <div id="app">
        <router-view v-if="isRouterActive"/>
    </div>
</template>
<script>
    export default{
        name: 'App',
        provide:{
            retrun{
                reload:this.reload
            }
        },
        data(){
            return{
                isRouterActive: true
            }
        },
        methods:{
            reload(){
                this.isRouterActive = false;
                this.$nextTick(()=>{
                     this.isRouterActive = true;
                })
            }
        }
    }
</script>

```

通过声明reload方法，控制isRouterAlice属性true or false 来控制router-view的显示或隐藏，从而控制页面的再次加载

2.在需要当前页面刷新的页面中注入App.vue组件提供（provide）的 reload 依赖，然后直接用this.reload来调用就行

```js
export defalut{
    inject:['reload'],
    data(){
        return{

        }
    },
    methods:{
        reloadPage(){
            this.reload();
        }
    }
}
```

# keep-alive

官网介绍：

`<keep-alive>` 包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。和 `<transition>` 相似，`<keep-alive>` 是一个抽象组件：它自身不会渲染一个 DOM 元素，也不会出现在父组件链中。

当组件在 `<keep-alive>` 内被切换，它的 activated 和 deactivated 这两个生命周期钩子函数将会被对应执行。