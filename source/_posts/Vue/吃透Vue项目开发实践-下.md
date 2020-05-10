---
title: Vue开发实践(下)
tags: 'Vue'
categories: 'Vue'
date: 2020-05-10 17:30:23
---

[原文地址](https://juejin.im/post/5e1eb1dff265da3e354ea2d0)   

# 前言
前面两篇文章总结了 Vue 开发的大部分技巧和内容，最后一篇文章来对它进行一个收尾
这篇文章我们来谈谈一些 Vue 理解和实践要求高一点的问题

首先是生命周期这一块内容，随着实践越多它的意义越大，理解也越深刻

mixin 功能强大，对代码复用组织都有很高的要求，算是 Vue 后期发力的高级技巧

服务端渲染可能是学习 Vue 最后一块阵地了，对于 SPA 框架的一个里程碑

最后，总结一下我在使用 Vue 中使用的技巧和经验

# 概览

![概览](https://user-gold-cdn.xitu.io/2020/3/11/170c9c19f1dd22b7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# Vue 生命周期

## 什么是 Vue 生命周期？

Vue 生命周期大概就是：一个从 Vue 实例的创建到组件销毁的一个的过程。

具体情况下，我们分为几个核心的阶段，并且每个阶段都有一套钩子函数来执行我们需要的代码。

### 生命周期阶段与钩子

我们整理分类一下这些生命周期钩子，为了记忆方便分为 4 大核心阶段：

方便读者记忆，这里尽量使用图示：
![图示](https://user-gold-cdn.xitu.io/2020/3/11/170c9c27264b6562?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

可以看到每一个阶段中的钩子命名都很好记忆，阶段开始前使用 `beforeXxx`，阶段后结束后使用`xxxed`

除这 8 个核心钩子，另外还有 3 个新增功能型钩子，目前总共是 11 个钩子 顺带提一下这 3 个钩子的功能

1. 组件缓存，`activated` 与 `deactivated`，这两个钩子也是一对的，分别表示被 keep-alive 缓存的组件激活和停用时调用。
2. 组件错误捕获，`errorCaptured`，对组件中出现对异常错误进行处理，使用较少。

### Vue 源码基本流程

对于上面那张图的理解，我们需要对 Vue 源码进行梳理，才能真正的理解。大概根据现有的源码，我梳理了一下大致的流程：

![源码](https://user-gold-cdn.xitu.io/2020/3/11/170c9c2cc22b99c2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们可以清楚的看到，从 Vue 实例创建、组件挂载、渲染的一些过程中，有着明显的周期节点。

### 简化文字图解

对于上面那张图的理解，我们需要对 Vue 源码进行梳理，才能真正的理解。大概根据现有的源码，我梳理了一下大致的流程：

![图解](https://user-gold-cdn.xitu.io/2020/3/11/170c9c2cc22b99c2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 实践验证一下生命周期

### 提出问题

下面我们提出一些问题：

1. 什么时期创建 el ？
2. 什么时期挂载 data ？
3. 什么时期可以访问 dom ？
4. 什么情况下组件会更新？更新是同步更新还是异步更新？
5. 什么情况下组件会被销毁？
6. 销毁组件后，还可以访问哪些内容？

### 编写代码

1. 首先写一个小 demo，打印关键组件信息

```html
<template>  
<div>    
<div class="message">      
{{message}}    
</div>  
</div>
</template>
<script>
export default {  
    data() {    
        return {      
            message: '1'    
        }
    },  
    methods: {    
        printComponentInfo(lifeName) {      
            console.log(lifeName)      
            console.log('el', this.$el)      
            console.log('data', this.$data)      
            console.log('watch', this.$watch)    
        }  
    }
}
</script>
```

2. 增加核心的 8 个生命周期钩子，分别调用打印方法

```js
  // ...  
  beforeCreate() {    this.printComponentInfo('beforeCreate')  },  
  created() {    this.printComponentInfo('created')  },  
  beforeMount() {    this.printComponentInfo('beforeMount')  },  
  mounted() {    this.printComponentInfo('mounted')  },  
  beforeUpdate() {    this.printComponentInfo('beforeUpdate')  },  
  updated() {    this.printComponentInfo('updated')  },  
  beforeDestroy() {    this.printComponentInfo('beforeDestroy')  },  
  destroyed() {    this.printComponentInfo('destroyed')  },  
  // ...
```

### 创建阶段

![创建](https://user-gold-cdn.xitu.io/2020/3/10/170c25e769e9f94e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

beforeCreate 中methods中方法直接报错无法访问，直接访问 el 和 data 后

![创建](https://user-gold-cdn.xitu.io/2020/3/10/170c26347c3622f5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

发现只能访问到 watch， el 和 data 均不能访问到

created 时期 el 无法访问到，但是可以访问到 data 了

### 挂载阶段

![挂载](https://user-gold-cdn.xitu.io/2020/3/10/170c2679f7e956d2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

`beforeMount` 中可以访问 data 但是仍然访问不到 el

`mounted` 中可以访问到 el 了

> 首次加载页面，更新阶段和销毁阶段到钩子都未触发

### 更新阶段

```js
this.message = this.message + 1
```

如果增加在 created 阶段，发现 update钩子仍然未触发，但是 el 和 data 的值都变成了 2

如果增加在 mounted 阶段，发现 update钩子此时触发了

![更新](https://user-gold-cdn.xitu.io/2020/3/10/170c2727250a6c8c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### 销毁阶段

怎样触发销毁的钩子呢？ 大概有这几种方法

* 手动调用 $destory
* v-if 与 v-for 指令，（v-show 不行）
* 路由切换和关闭或刷新浏览器

![路由切换](https://user-gold-cdn.xitu.io/2020/3/10/170c2ef6a6785ddc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


## 生命周期钩子常见使用的场景

### beforeCreate 谨慎操作 this

`beforeCreate` 无法访问到 this 中的 `data、method`

### 请求应放在 created 钩子中

created 可以访问 this，但无法访问 dom，dom 未挂载

### 操作 DOM 代码应放在 mounted 钩子中

mounted 已经挂载 dom，可以访问 this

> 生命周期相关demo 代码见[github-lifecycle-demo](生命周期相关demo 代码见github-lifecycle-demo)

# 理解并合理使用 mixin

## 什么是 mixin（混入）

> 当组件使用混入对象时，所有混入对象的选项将被“混合”进入该组件本身的选项 大致原理就是将外来的组件、方法以某种方式进行合并。合并的规则有点像继承和扩展。

当组件和混入对象含有**同名选项**时，这些选项将以恰当的方式进行“合并”

我们看一下一个组件里面有哪些东西是可以合并的

`data、methods、computed、directives、components` 生命周期钩子

没错这些都可以混入

```js
import demoMixin form '@/mixins/demo'
export default {    
    mixins: [demoMixin]
}
```

这样看来，很多页面重复的代码我们都可以直接抽取出来
或者是封装成一个公共的 `mixin`
比如我们做 H5 页面，里面很多短信验证的逻辑固有逻辑，但是需要访问到 this。使用工具函数肯定不行。
这时候就可以考虑使用 `mixin`，封装成一个具有响应式的模块。供需要的地方进行引入。

## mixin 规则

首先是**优先级**的问题，当重名选项时选择哪一个为最后的结果

默认规则我这里分为 3 类

* 1. data 混入: 以当前组件值为最后的值
* 2. 生命周期钩子: 保留所有钩子，先执行 mixins 的，后执行当前组件的
* 3. methods、computed、directives、components 这种健值对形式，同名key，统统以当前组件为准

当然如果想改变规则，也可以通过配置来改变规则

```js
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
    // 返回合并后的值
}
```

## mixin 的好处

我们知道 `Vue` 最能复用代码的就是组件。一般情况，我们通过 `props` 来控制组件的，将原有组件封装成 `HOC` 高阶组件。而控制 `props` 的生成不一样的功能的代码还是写在基础组件里。

```html
<template lang="pug">  
div.dib    
van-button.btn(      
    @click="$emit('click')"      
    :class="getClass" v-bind="$attrs"      
    :style="{'width': size === 'large' ? '345px': '', 
    'backgroundColor': getBgColor, 
    borderColor: getBgColor, color: getBgColor}"
)      
slot
</template>
<script>
import { Button } from 'vant'
import Vue from 'vue'
import { getColor } from '@/utils'
Vue.use(Button)
export default {  
    name: 'app-button',  
    props: {    
        type: {      
            type: String,      
            default: 'primary'    
        },    
        theme: {      
            type: String,      
            default: 'blue'    
        },    
        size: {      
            type: String,      
            default: ''   
        }  
    }
}
```

以这个组件为例，我们还是**通过公共组件内部的逻辑，来改变组件的行为**。

但是，使用 mixin 提供了另一个思路。我们写好公共的 mixin，每一个需要使用 mixin 的地方。我们进行扩展合并，不同与公共 mixin 的选项我们在当前组件中进行自定义，也就是扩展。**我们新的逻辑是写在当前组件里面的，而非公共 mixins 里**。

画个图理解一下：

![画个图](https://user-gold-cdn.xitu.io/2020/3/10/170c3937144fd11f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 最后总结一下 mixin 的优点及缺点

**优点**

* 复用代码，公共逻辑抽离
* 可以访问 this, 可以操作响应式代码
* mixins 相对与组件来说，更像扩展

**缺点**

* 千万不能滥用全局 mixins 因为会影响所有多子组件
* 由于 mixins 的合并策略固有影响，可能在一些极端情况达不到你想要的效果。

比如：我已经存在一个 `mixins`，我的页面里面也有一些方法，我要引入`mixins` 就要做很多改动，来保证我的代码按我的需要运行。

页面 data 有一个 `message` 值，`mixins` 里面同样有一个。

按照默认规则，`mixins` 里面的 `message` 会被页面里面 `message` 覆盖。但是这两个 `message` 可能代表的意义不一样，都需要存在。那么我就需要改掉其中的一个，如果业务太深的话，可能这个 `message` 没那么好改。

有时候需要考虑这些问题，导致使用 `mixins` 都会增加一些开发负担。当然也是这些问题可以使用规范来规避。

# 理解并使用 SSR

> SSR 是 Serve Side Render 的缩写，翻译过来就是我们常说的服务端渲染

## 简单归纳 SSR 存在的原因

1. SPA 框架 SEO 的解决方案
2. 提升首屏加载速度

但是它还存在以下问题

* 有些生命周期钩子无法使用（之前提到的 activated 和 deactivated 等等）
* 额外很多的配置
* 服务器资源的需求

总得来说，SSR 是必要的但不是充分的，SPA 的 SEO 现在没有更好的方案，有这方面强烈需求的网站来说，SSR 确实很有必要

## SSR 原理

![SSR 原理](https://user-gold-cdn.xitu.io/2020/3/10/170c3ed6607a605c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

通过上面图我们可以得大致几点内容

* 通用业务代码只写一套。
* 通过暴露客户端和服务端两个入口，对 Vue 实例进行访问。
* 通过构建工具打成两个包，服务端包通过 node 服务器渲染给浏览器访问。客户端和原来一样通过访问资源获取。

## 实现SSR 有 3 种方式

1. 根据官网教程搭建，一步一步搭建SSR， 这里有具体教程，略微有点麻烦，但是能够体验搭建的过程，更加深入细节。使用于练习，和现有项目的改造
2. 使用demo改造，开源 demo，方便省时，适用代码参考学习 [vue-ssr-demo](https://github.com/vuejs/vue-hackernews-2.0)
3. 使用nuxt，预设了 SSR 和预渲染，适用于需要 SSR 的新项目。

**分别尝试用这 3 种方式搭建 SSR**

## 五步简单理解SSR（根据官网教程，具体完操作查看整教程和demo）

*这里主要加深理解，vue-cli3+ 实现基本 SSR*

### 第一步：安装依赖
* vue-server-renderer （核心依赖，版本必须与 vue 版本一致）
* webpack-merge（用于webpack配置合并）
* webpack-node-externals （用于webpack配置更改）
* express (用于服务端渲染)

### 第二步：建立入口，并改造改造

分为 2 个入口，将 `main.js` 定为通用入口， 并额外增加`entry-client.js` 和 `entry-serve.js` 两个

1.改造主要入口，创建工厂函数

```js
// main.js
import Vue from 'vue'
import App from './App.vue'
import { createRouter } from "./router"
// app、router
export function createApp () {  
    const router = createRouter()  
    const app = new Vue({    
        router,    
        render: h => h(App)  
    })  
    return { 
        app, router 
    }
}
```

2.客户端入口

```js
// client.js
import { createApp } from './main'
// 客户端特定引导逻辑
const { app } = createApp()
app.$mount('#app')
```

3.服务端入口

```js
// serve.js
import { createApp } from "./main";
export default context => {  
    // 因为有可能会是异步路由钩子函数或组件，所以我们将返回一个 Promise  
    return new Promise((resolve, reject) => {    
        const { app, router } = createApp();    
        // 设置服务器端 router 的位置    
        router.push(context.url);    
        // 等到 router 将可能的异步组件和钩子函数解析完    
        router.onReady(() => {      
            const matchedComponents = router.getMatchedComponents();      
            // 匹配不到的路由，执行 reject 函数      
            if (!matchedComponents.length) {        
                return reject({          
                    code: 404        
                });      
            }      
            // Promise 应该 resolve 应用程序实例，以便它可以渲染      
            resolve(app);   
            
        }, reject);  
    });
};
```

### 第三步：改造 vue.config 配置

```js
const VueSSRServerPlugin = require("vue-server-renderer/server-plugin");
const VueSSRClientPlugin = require("vue-server-renderer/client-plugin");
const nodeExternals = require("webpack-node-externals");
const merge = require("webpack-merge");
const TARGET_NODE = process.env.WEBPACK_TARGET === "node";
const target = TARGET_NODE ? "server" : "client";
module.exports = {  
    configureWebpack: () => ({    
        entry: `./src/entry-${target}.js`,    
        devtool: 'source-map',    
        target: TARGET_NODE ? "node" : "web",    
        node: TARGET_NODE ? undefined : false,    
        output: {      
            libraryTarget: TARGET_NODE ? "commonjs2" : undefined    
        },    
        externals: TARGET_NODE      
        ? nodeExternals({          
            whitelist: [/\.css$/]        
        }): undefined,    
        optimization: {          
            splitChunks: undefined    
        },    
        plugins: [
            TARGET_NODE ? new VueSSRServerPlugin() : new VueSSRClientPlugin()]  
        }), 
         
        //...
};
```

### 第四步：对路由 router 改造

```js
export function createRouter(){  
    return new Router({    
        mode: 'history',    
        routes: [    
            //...    
        ]  
    })
}
```

### 第五步：使用 express 运行服务端代码

这一步主要是让 node 服务端响应 HTML 给浏览器访问

```js
const Vue = require('vue')
const server = require('express')()
const renderer = require('vue-server-renderer').createRenderer()
server.get('*', (req, res) => {  
    const app = new Vue({    
        data: {      
            url: req.url    
        },    
        template: `<div>访问的 URL 是： {{ url }}</div>`  
    })  
    renderer.renderToString(app, (err, html) => {    
        if (err) {      
            res.status(500).end('Internal Server Error')      
            return    
        }    
        res.end(`      
            <!DOCTYPE html>      
            <html lang="en">       
            <head><title>Hello</title></head>        
            <body>${html}</body>      
            </html>`)  
    })
})
server.listen(8080)
```

## nuxt 体验

简单几步体验下 nuxt

安装后

![nuxt](https://user-gold-cdn.xitu.io/2020/3/11/170c7cbe963be44c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

简单看了一下源码，nuxt 把我们之前提到的重要的改造，全部封装到 .nuxt 文件夹里面了\

![启动](https://user-gold-cdn.xitu.io/2020/3/11/170c7cc50e09b590?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

跑一下 dev 发现有两个端，一个 clinet 端，一个 server 端

![demo](https://user-gold-cdn.xitu.io/2020/3/11/170c7cc93fe6230b?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

最后查看一下效果，整个过程挺丝滑的。目录结构也比较符合我的风格，新项目需要 SSR 会考虑使用 nuxt

## 使用 webpack 插件预渲染

解决 SEO 问题是不是只有 SSR 呢？其实预渲染也能做到，首先

### 区别使用 SSR 和预渲染

* 服务端渲染解决的问题，不仅只是把 HTML 页面给浏览器，更重要的是处理动态逻辑和 JS 代码后，将渲染后完整的 HTML 给浏览器，渲染的过程在服务端。
* 预渲染，是利用构建工具在 webpack 中生成静态的 HTML，直接给浏览器，渲染的过程在本地。
* 预渲染插件里面提到两种不能使用：大量路由、动态内容

### 使用 prerender-spa-plugin 插件进行简单预渲染

1. 安装 prerender-spa-plugin

```js
yarn prerender-spa-plugin
```

2. 修改 webpack 配置，比较简单就能完成配置

```js
const path = require('path')
const PrerenderSPAPlugin = require('prerender-spa-plugin')
const Renderer = PrerenderSPAPlugin.PuppeteerRenderer
module.exports = {  
    plugins: [    
        //...    
        new PrerenderSPAPlugin({      
            staticDir: path.join(__dirname, 'dist'),      
            outputDir: path.join(__dirname, 'prerendered'),      
            indexPath: path.join(__dirname, 'dist', 'index.html'),      
            routes: [ '/', '/about', '/some/deep/nested/route' ],      
            postProcess (renderedRoute) {        
                renderedRoute.route = renderedRoute.originalPath        
                renderedRoute.html = renderedRoute.html.split(/>[\s]+</gmi).join('><')        
                if (renderedRoute.route.endsWith('.html')) {          
                    renderedRoute.outputPath = path.join(__dirname, 'dist', renderedRoute.route)        
                }        
                return renderedRoute      
            },      
            minify: {        
                collapseBooleanAttributes: true,        
                collapseWhitespace: true,        
                decodeEntities: true,        
                keepClosingSlash: true,        
                sortAttributes: true      
            },      
            renderer: new Renderer({        
                inject: {          
                    foo: 'bar'        
                },        
            maxConcurrentRoutes: 4  
        ]
}
```

# Vue 开发技巧总结

> 从 5 个大的角度来提升开发效率和体验，代码美观和代码质量，用户体验

![技巧](https://user-gold-cdn.xitu.io/2020/3/11/170c9c3a14c4cf39?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 5 大角度提升

### 代码复用

* 组件化开发，代码效率 * n
* 使用 mixins 抽离公共逻辑，代码效率 * n
* 工具化函数、使用 filter 编码效率+
* sass 复用 css，编码体验、效率+

### 代码质量

* 代码静态检查 eslint + prettier，代码风格+、基础语法错误-
* 数据类型控制 typescript，代码质量+
* 前端测试 test，代码质量+

### 代码优化

* 合理使用 vue，渲染性能+
* 合理使用 vuex 减少请求，使用图片懒加载，加载性能+
* 合理使用函数组件，组件性能+
* 合理骨架屏、路由过渡，用户体验+

### 开发效率

* 使用更新的脚手架 vue-cli4，webpack 配置效率+
* 使用配置好的脚手架模版 vue-h5-template，vue 配置效率+
* 使用更简洁的模版 pug，HTML 编写效率+
* 使用更强大的 css 编写 sass，CSS 编写效率+
* 使用模拟数据 mock，脱离后端开发效率+
* 开源组件封装 HOC，组件开发，页面编写效率+

### 瓶颈解决
* 路由history使用，服务端配置相关，URL美观+
* 解决SEO与首屏加载、服务端渲染 SSR 基本解决
