---
title: vue.config.js基本配置
date: 2020-04-01 16:06:12
tags: "Vue"
categories: "Vue"
---

```js
// vue.config.js 基本配置方法
module.exports = {
  // 项目部署的基础路径
  // 我们默认假设你的应用将会部署在域名的根部，
  // 比如 https://www.my-app.com/
  // 如果你的应用时部署在一个子路径下，那么你需要在这里
  // 指定子路径。比如，如果你的应用部署在
  // https://www.foobar.com/my-app/
  // 那么将这个值改为 `/my-app/`
  // 基本路径 baseURL已经过时
  publicPath: './',
  
	// 打包项目时构建的文件目录，用法与webpack本身的output.path一致
  outputDir: 'dist', 
  
  // 静态资源目录 (js, css, img, fonts)
  assetsDir: 'assets',
  
  // eslint-loader 是否在保存的时候检查，编译不规范时，设为true在命令行中警告，若设为error则不仅警告，并且编译失败
  lintOnSave: true,
  
  // 调整内部的 webpack 配置。查阅 https://github.com/vuejs/vue-docs-zh-cn/blob/master/vue-cli/webpack.md
  chainWebpack: () => {},
  configureWebpack: () => {},
  
  // vue-loader 配置项 https://vue-loader.vuejs.org/en/options.html
   vueLoader: {},
  
  // 生产环境是否生成 sourceMap 文件，默认true，若不需要生产环境的sourceMap，可以设置为false，加速生产环境的构建
  productionSourceMap: true,
  
  // css相关配置
  css: {
   // 是否使用css分离插件 ExtractTextPlugin，采用独立样式文件载入，不采用<style>方式内联至html文件中
   extract: true,
   // 是否在构建样式地图，false将提高构建速度
   sourceMap: false,
   // css预设器配置项
   loaderOptions: {},
   // 启用 CSS modules for all css / pre-processor files.
   // 这个选项不会影响 `*.vue` 文件
   modules: false
  },
  
  // 在生产环境下为 Babel 和 TypeScript 使用 `thread-loader`
  // 在多核机器下会默认开启。
  parallel: require('os').cpus().length > 1,
  
  // 是否启用dll See https://github.com/vuejs/vue-cli/blob/dev/docs/cli-service.md#dll-mode
  dll: false,
  
  // PWA 插件相关配置 see https://github.com/vuejs/vue-cli/tree/dev/packages/%40vue/cli-plugin-pwa
  pwa: {},
  
  // webpack-dev-server 相关配置
  devServer: {
   open: process.platform === 'darwin',
   host: '0.0.0.0',//如果是真机测试，就使用这个IP
   port: 1234,
   https: false,
   hotOnly: false,
   proxy: null, // 设置代理
   // proxy: {
   //     '/api': {
   //         target: '<url>',
   //         ws: true,
   //         changOrigin: true
   //     }
   // },
   before: app => {}
  },
  // 第三方插件配置
  pluginOptions: {
   // ...
  }
 }
```