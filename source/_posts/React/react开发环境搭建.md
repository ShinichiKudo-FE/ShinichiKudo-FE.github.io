---
title: react开发环境搭建
date: 2018-02-28 10:18:35
tags: "React"
categories: "React"
---

# 使用creat-react-app搭建基于webpack的react开发环境

--------------

[facebook官方文档](https://github.com/facebook/create-react-app)
**Quick Start**:

```javascript
npm install -g create-react-app       //安装create-react-app，yarn命令(需要安装yarn) yarn add create-react-app

create-react-app myapp  //使用命令创建react应用 myapp为文件名称

cd myapp     //进入文件目录

npm start //yarn start  启动应用

npm test or yarn test //测试

npm run build or yarn build //打包编译
```

上述命令即可快速创建react开发环境

打开http://localhost:3000/ 查看
<!-- more -->

## 环境配置介绍：

### 项目结构：
生成项目后，脚手架为了“优雅”... ...隐藏了所有的webpack相关的配置文件，此时查看myapp文件夹目录，会发现找不到任何webpack配置文件。执行以下命令：
```javascript
npm run eject
```

再查看myapp 文件夹，可以看到完整的项目结构：
```javascript
myapp/
    node_modules/
    package.json
    .gitignore
    config/
        paths.js
        polyfills
        env.js
        webpack.config.dev.js
        webpack.config.prod.js
    public/
        index.html   / 入口html文件 /
    scripts
        **build.js**
        **start.js**
        test.js
    src/
        App.js
        index.js    / 主入口文件 /
```
加粗文件为主要配置文件。

二、项目配置介绍
此处需要有npm、webpack的基础知识，充电传送门：[webpack学习教程](http://www.jianshu.com/p/42e11515c10f)

查看package.json文件的scripts，
```javascript
"scripts": {

    "start": "node scripts/start.js",

    "build": "node scripts/build.js",

    "test": "node scripts/test.js --env=jsdom"

},
```

可知，运行时是直接执行scripts文件目录下的js文件。
其中**start.js**为开发环境，**build.js**为打包脚本。

### 开发环境，代理请求
查看**start.js**, Facebook基本为每项配置都写了详尽的注释，
**start.js**脚本启动了dev-server, 这里需要了解的是

```javascript
function addMiddleware(devServer){
    ... ...
    这个函数调用http-proxy-middleware模块，将代理请求，代理地址在package.json中添加

}
```

在package.json中添加字段proxy，开发环境下dev-server将会自动转发请求：
```javascript
"proxy": "http://aaa.bbb.com"
```

### SASS、LESS等CSS预处理器配置
Facebook官方在create-react-app升级到某一版本，突然丢掉了默认对sass、less等预处理器的支持，官方文档说明

于是，只能自己动手，在config目录下，**webpack.config.dev.js** 和 **webpack.config.prod.js**文件，没有抽出 loader、postcss之类一般共用的配置，于是，在两个文件夹都要一起配置，也可以抽出共用部分，以便维护。

这里以**webpack.config.dev.js**举例，**webpack.config.prod.js**一样配置即可：

SASS-loader：
1、命令行，在当前目录执行：
```javascript
npm install --save node-sass-chokidar //or yarn add node-sass-chokidar
```
然后在package.json修改为
```javascript
"scripts": {
   "build-css": "node-sass-chokidar src/ -o src/",
   "watch-css": "npm run build-css && node-sass-chokidar src/ -o src/ --watch --recursive",
     "start": "react-scripts start",
     "build": "react-scripts build",
     "test": "react-scripts test --env=jsdom",
```
把index.css改为index.scss 执行 
```javascript
npm run watch-css
```
就可以发现sass文件被编译成了css文件（自动生成的css文件所以引用的部分不作修改）

如果你发现在执行**watch-css** 命令时 不能同时执行npm start,可以采用另外一种方式：
```javascript
npm install --save npm-run-all //yarn add npm-run-all（ 这是watch-css和build-css的合集）
```
同样修改package.json中的
```javascript
 "scripts": {
    "build-css": "node-sass-chokidar src/ -o src/",
    "watch-css": "npm run build-css && node-sass-chokidar src/ -o src/ --watch --recursive",
    "start-js": "node scripts/start.js",
    "start": "npm-run-all -p watch-css start-js",
    "build-js": "node scripts/build.js",
    "build": "npm-run-all build-css build-js",
    "test": "node scripts/test.js --env=jsdom"
  },
```
这样就可以只需执行npm start 编辑查看了

2、找到webpack.config.dev.js文件中 loaders中的第一项exclude（值为数组），排除scss文件，否则将被'url-loader'捕获。
```javascript
{

exclude: [
    /\.html$/,
    /\.(js|jsx)(\?.*)?$/,
    /\.css$/,
    /\.json$/,
    /\.svg$/,
    /\.scss$/     //....新增项!
]
```

3、loaders新增一项：
```javascript
{
    test: /\.scss$/,
    loader: 'style!css!postcss!sass?outputStyle=expanded'
},
```

至此，SASS文件就可以正常打包了（此处针对SCSS文件，如用到SASS，可将SCSS的正则修改下），LESS文件类似，不再赘述。

## 使用and-mobile UI工具 可以快速开发react应用
[and-mobile入口](https://mobile.ant.design/index-cn)

### antd-mobile的引入及配置
在命令行执行：
```javascript
npm install antd-mobile --save
```

**按需引入**

为减少打包后体积以及方便书写，可用babel-plugin-import插件，配置后引入模块可如下：
```javascript
import {Picker} from 'antd-mobile';
```
自动引入CSS和JS，无需再引入整个antd-mobile的整个CSS文件

使用如下：
```javascript
npm install babel-plugin-import --save-dev
```

安装完毕后，在根目录新建文件，命名: .babelrc.js
```javascript
{
  "presets": [
    "es2015",
    "react"
  ],
  "plugins": [
    [
      "import",
        {
          "libraryName": "antd-mobile",
          "style": "css"
        }
      ]
    ]
}

```

在文件内输入以上内容，为babel的配置。

然后！！！最重要的一步，**把package.json中的babel配置给删掉，尤其是：react-app！！！**

webpack.config.dev.js和webpack.config.prod.js中，都需要为resolve的extensions新增一项'**.web.js**'

antd-mobile的web版的文件后缀为.web.js ...

## 基于create-react-app的打包后文件根路径修改 
找到react-script模块文件夹config下面  paths.js 或
node_modules\react-scripts\config\ paths.js
45行到 49行
```javascript
function getServedPath(appPackageJson) {
  const publicUrl = getPublicUrl(appPackageJson);
  const servedUrl =
    envPublicUrl || (publicUrl ? url.parse(publicUrl).pathname : './'); // 配置文件跟路径'/'前面加.
  return ensureSlash(servedUrl, true);
}
```

## react段位排行要求[知乎大神总结](https://www.zhihu.com/people/mo-ran-biao-82/activities)

### 倔强青铜
初入峡谷的初始段位，默认召唤师已经有了ES6，nodejs的基础使用create-react-app建立React开发环境学习React世界里的基本玩法，例如组件化，JSX，事件监听，内部state，组件的props、生命周期函数等这篇文章主要介绍React青铜升白银需要的基础知识，看完你就白银啦
### 秩序白银
到了白银段位，基本都是有了基本的操作，不会出现呆萌的站在地方塔下被打死的情况了我们需要买一个皮肤来提升页面美观并且多加练习学习使用蚂蚁金服ant-design的使用
### 荣耀黄金
到了这个阶段，召唤师对React有了基本的认识，想进一步的提升段位，我们需要提高自己的大局观学习使用React-Router4来让我们有多面作战能力学会使用BrowserRouter，Router，Link等组件学会使用Redux和队友配合，修炼大局观了解单项数据流开发模式和Redux的各种概念,如dispatch,action,reducers使用react-redux更好的和Redux配合有大局观意识，上铂金也是很 easy 了
### 尊贵铂金
很多召唤师卡在铂金上不去，因为铂金想上钻石，需要了解更多的细节和原理React原理剖析对自己技能的原理有深刻的了解，上钻石必备
### 永恒钻石
钻石段位开始了征兆模式，召唤师的技能池要足够深才能更进一步，对自己擅长技能的理解也要更深刻Redux中间件机制，实现自己的中间件常见的React 性能优化方式服务端渲染Redux之外其他的数据解决方案如mobx，dva
### 至尊星耀
这个段位已经是独当一面的强者了，目标仅限于 React 这个库很难更近一层，需要了解更底层的原理Redux原理剖析+实现自己的ReduxReact-Router+实现自己的React-RouterWebppack工作机制和原理剖析Babel工作机制和原理剖析
### 最强王者 
达到最强王者已经是顶尖召唤师，在整个 React 峡谷都是鼎鼎大名的存在，听说上面还有传说中的**荣耀王者**的段位，我这辈子是达不到了，也没法教你们了，囧这个阶段，我只推荐《程序员健康指南》一本书，保持身心健康，成为荣耀王者是早晚的事
[原文链接](https://www.jianshu.com/p/5e6c620ff4d6)

