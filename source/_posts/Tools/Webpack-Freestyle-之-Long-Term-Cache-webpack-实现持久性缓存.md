---
title: Webpack Freestyle 之 Long Term Cache(webpack 实现持久性缓存)
date: 2019-07-31 14:36:29
tags: "Webpack"
categories: "Tools"
---

[原文地址](https://zhuanlan.zhihu.com/p/27710902)

## How Browser Cache Works

首先，我们要搞清楚浏览器缓存是怎么工作的。 那么，就让我画一张图来告诉大家吧，嘻嘻。

![pic](https://pic2.zhimg.com/80/v2-5babc40038bcc8add9c5f8bcb717c7b5_hd.png)

* 浏览器: 我需要 foo.js
* 服务器: 让我找找。找到了，给你，缓存有效期为 1 年。
* 浏览器: 好，我把他缓存到磁盘里。

过了 2 天，

![pic1](https://pic4.zhimg.com/80/v2-b5e4a95482e0008758b94ec96c0f60a3_hd.png)

浏览器: 我需要 foo.js ，在缓存里找到了。缓存还有效，那直接读缓存。
用户：哇塞，这网页秒开啊。

又过了 2 天，foo.js 的代码更新了。（内容从 Hello world 变成了 Goodbye world ）

![pic2](https://pic2.zhimg.com/80/v2-680f699d7e09991af4f6bf260559aef1_hd.png)

浏览器：我需要 foo.js ，在缓存里找到了。缓存还有效，那直接读缓存。
产品经理：？？？这页面怎么跟以前一样啊？

很尴尬。foo.js 明明更新了，但是浏览器还是读取在缓存中旧的 foo.js ，原因是我们用了缓存，不用缓存就没这事儿了。?

解决办法嘛，当然有的。比如每次利用缓存之前，`先向服务器确认文件是否有更新，有更新则使用新的否则读缓存`。还有一种方法是`把缓存破坏掉，也就是下面要说的*Cache Busting Technique* ` 

## Cache Busting Technique

因为 foo.js 的代码变化了，但是他的缓存还没失效，此时浏览器还是会读取以前的缓存了的 foo.js ，并不会去服务器下载最新的。这显然不是我们想要的，怎么办呢？我们需要破坏缓存（ Cache Busting ）。

破坏缓存并不是禁止缓存，而是换一种方式让缓存失效。比如：

1.修改文件的名字：foo.js -> foo.v2.js
2.修改文件的路径：/static/foo.js -> /static/v2/foo.js
3.加 query string : foo.js -> foo.js?v=qwer

我们下面将采用第一种方法，也就是修改文件的名字。我们把更新后的 foo.js 的文件名改成 foo.v2.js 。这样，浏览器就不会去读取缓存里的旧的 foo.js ，而是向服务请请求 foo.v2.js ，如下图所示：

![pic4](https://pic2.zhimg.com/80/v2-103471cbe5d028bc9e1122db6e4243b9_hd.png)

那么，假设我们现在有很多很多的静态文件，然后每次需要更新很多很多的文件，那是不是要手动地一个一个地修改文件的名字呢？我们的理想当然是：哪个文件更新了，就**自动**地生成一个新的文件名。

另外，如果我们打包出来的静态文件只有一个单独的 JavaScript 文件 app.js ，那么每次改动一点代码，app.js 的文件名肯定都会变。但实际上，我只改动了某个模块的代码（其他模块并没有修改），就破坏了其他模块的缓存，这显然没有充分利用到缓存啊。我们的想法是：哪个模块更新了破坏他的缓存，没更新的模块继续利用缓存。

这个时候，我们就需要用到 webpack 的 code splitting（如果还不会的话，可以阅读 [Webpack 大法之 Code Splitting](https://zhuanlan.zhihu.com/p/26710831?refer=ElemeFE) ）。把整个 App 分成一个个 chunk ，然后哪个 chunk 发生改变，我就破坏他的缓存；没有更新的 chunk ，则继续利用缓存。这样一来，我们就把缓存的作用发挥到淋漓尽致～

所以，`code splitting` 的作用除了”减少文件大小”之外，还能更充分地利用缓存。所以，下面就让我们用 webpack 来实现持久性缓存吧。

## Webpack & Caching

首先，把我们的 [demo 项目](https://link.zhihu.com/?target=https%3A//github.com/lyyourc/webpack-code-splitting-demo)（已经实现了 code splitting ）下载并安装好依赖。

接着，修改 webpack 配置文件，给我们打包后的静态文件生成随机的唯一的名字。（ changed files ）

```js
// webpack.config.js
module.exports = {
  output: {
    //...
    filename: '[name].[chunkhash:8].js',
    chunkFilename: '[name].[chunkhash:8].chunk.js',
    //...
  },
}
```

我们使用了 `[chunkhash]` 这个占位符，并且为了更好地分辨和展示 demo ，我们截取了他的前 8 个字符 `[chunkhash:8]`，但是在实际生产中我们不要那么做！

好咯，现在来看看我们的打包后的文件：

```js
                    Asset            Size    Chunk Names
common-in-lazy.fa79d198.chunk.js    11.6 kB  common-in-lazy
    used-twice.c2c4927c.chunk.js    17.1 kB  used-twice
        Photos.28d663ec.chunk.js    8.57 kB  Photos
         Emoji.d3ea8991.chunk.js    1.15 kB  Emoji
                 app.724a238a.js    2.53 kB  app
              vendor.05be8f94.js     104 kB  vendor
```

那么，现在我们来修改一下 App.vue ，添加个 `<footer>` 标签（ changed files ） :

```html
<template>
<div id="app">
  <!-- old codes -->
  <footer> A Footer </footer>
</div>
</template>
```
<!-- more -->
此时的打包变成了：

```js
                        Asset       Size  Chunk Names
common-in-lazy.fa79d198.chunk.js    11.6 kB  common-in-lazy
    used-twice.c2c4927c.chunk.js    17.1 kB  used-twice
        Photos.28d663ec.chunk.js    8.57 kB  Photos
         Emoji.d3ea8991.chunk.js    1.15 kB  Emoji
                 app.fdc2eedb.js    2.57 kB  app
              vendor.b611a5da.js     104 kB  vendor
```

注意到，我们的 app chunk 的 hash 从 724a238a 变成了 fdc2eedb ，这是我们所希望看到的东西。但是，与此同时 vendor chunk 的 hash 也变了（05be8f94 -> b611a5da）。然而，我们并没有修改 vendor chunk 的代码，为什么他的 hash 也变了呢？?

原因是 `vendor chunk` 里面包含了 `webpack` 的 `runtime` 代码（用来解析和加载模块之类的运行时代码）：

解决办法就是把 webpack 的 runtime 代码提取出来（ changed files ）：

```js
new webpack.optimize.CommonsChunkPlugin({ 
  name: ['manifast'] 
}),
```

把之前 App.vue 更新了的代码暂时去掉，也就是上面添加的 `<footer>` 标签去掉：

接着按照之前的，修改 App.vue ，添加 `` 标签（ changed files ）：

```html
<template>
<div id="app">
  <!-- old codes -->
  <footer> A Footer </footer>
</div>
</template>
```

而此时的打包：

```js
                        Asset       Size  Chunk Names
common-in-lazy.fa79d198.chunk.js    11.6 kB  common-in-lazy
    used-twice.c2c4927c.chunk.js    17.1 kB  used-twice
        Photos.28d663ec.chunk.js    8.57 kB  Photos
         Emoji.d3ea8991.chunk.js    1.15 kB  Emoji
                 app.fdc2eedb.js    2.57 kB  app
              vendor.3b70f9d8.js     103 kB  vendor
            manifast.1442e3f3.js    1.54 kB  manifast
```

很开心，此时只有 `app.js` 和 `manifast.js` 这 2 个 chunk 的文件名的 hash 发生了改变，vendor.js chunk 和其他 chunk 都没变，舒服。

但是，假如我们给 App.vue 随便引入一个模块的话，比如（ changed files ）：

```js
<script>
//...
import noop from './shared/utils.js'
</script>
```

而此时的打包：

```js
                        Asset       Size  Chunk Names
common-in-lazy.30b1e9b6.chunk.js    11.6 kB  common-in-lazy
    used-twice.9eccbe5a.chunk.js    17.1 kB  used-twice
        Photos.6096611c.chunk.js    8.57 kB  Photos
         Emoji.6208da60.chunk.js    1.15 kB  Emoji
                 app.4675a374.js    2.61 kB  app
              vendor.8b538297.js     103 kB  vendor
            manifast.25580296.js    1.54 kB  manifast
```

卧槽居然所有 chunk 的 hash 都发生了改变，这是为什么？

**原因是在 webpack 里每个模块都有一个 module id **，module id 是该模块在[模块依赖关系图](https://link.zhihu.com/?target=https%3A//webpack.js.org/concepts/dependency-graph/)里按顺序分配的序号，如果这个 module id 发生了变化，那么他的 chunkhash 也会发生变化。（不确定这里是否正确，希望大佬指出错误）

![pic5](https://pic4.zhimg.com/80/v2-5a4b6bef5809e00512873d481a3670e7_hd.png)

所以呢，我们需要用一种新的方式来计算 module id 。 **HashedModuleIdsPlugin** 这个插件，他是根据模块所在路径来映射其 module id ，这样就算引入了新的模块，也不会影响 module id 的值，只要模块的路径不改变的话。

修改我们的 webpack 配置。并且，去掉上面 App.vue 引入的 noop 模块。（ changed files ）

```js
// webpack.config.js

plugins: [
  new webpack.HashedModuleIdsPlugin(),
  // ...
],
```

可以看到，只有 app chunk 和 manifast chunk 的 hash 发生了改变，其他 chunk 不变所以他们的缓存就没被破坏。

## 总结


用 webpack 实现 long term cache ：

* 生成稳定的 hash 文件名
* 提取 webpack 的 runtime 代码
* code splitting

还有一些东西我们是没讲到的，比如 CSS 的 cache ，内联 manifast chunk 等等，就留给大家去探索咯。

最后需要注意的是，webpack 是允许其他 plugin 来修改 chunkhash 的，如果他们不能正确地处理的话，那么，假设你更新了代码，但是对应的 chunkhash 没变，并且此时缓存还没失效，就会导致线上的代码还是旧的，用户看到的还是以前的页面。因此，一定要特别注意 chunkhash 到底正不正确！！

希望本文可以帮助到大家，这样我会很开心的。(* *)