---
title: gulp配置及使用
date: 2018-02-24 13:58:31
tags: "Gulp"
categories: "Tools"
---
# gulp 入门指南
## 1.全局安装 gulp：
```javascript
$ npm install --global gulp
```

## 2.作为项目的开发依赖（devDependencies）安装：
```javascript
$ npm install --save-dev gulp
```

## 3. 在项目根目录下创建一个名为 gulpfile.js 的文件：
```javascript
var gulp = require('gulp');

gulp.task('default', function() {
  // 将你的默认的任务代码放在这
});
```
## 4. 运行 gulp：
```javascript
$ gulp
```
<!-- more -->

**五个gulp命令**:
>gulp.task(name[, deps], fn) 定义任务  name：任务名称 deps：依赖任务名称 fn：回调函数
gulp.run(tasks...)：尽可能多的并行运行多个task
gulp.watch(glob, fn)：当glob内容发生改变时，执行fn
gulp.src(glob)：置需要处理的文件的路径，可以是多个文件以数组的形式，也可以是正则
gulp.dest(path[, options])：设置生成文件的路径


## **package.json文件**
```json
{
  "name": "yunbaodan",
  "version": "1.0.0",
  "description": "2.0升级",
  "main": "gulpfile.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "zhang",
  "license": "ISC",
  "devDependencies": {
    "del": "^3.0.0",
    "gulp": "^3.9.1",
    "gulp-autoprefixer": "^4.0.0",
    "gulp-cache": "^1.0.1",
    "gulp-clean": "^0.3.2",
    "gulp-concat": "^2.6.1",
    "gulp-gh-pages": "^0.5.4",
    "gulp-htmlmin": "^3.0.0",
    "gulp-imagemin": "^4.0.0",
    "gulp-jshint": "^2.0.4",
    "gulp-less": "^3.3.2",
    "gulp-livereload": "^3.8.1",
    "gulp-minify-css": "^1.2.4",
    "gulp-notify": "^3.0.0",
    "gulp-postcss": "^7.0.0",
    "gulp-rename": "^1.2.2",
    "gulp-ruby-sass": "^2.1.1",
    "gulp-file-include": "^2.0.0",
    "gulp-sass": "^3.1.0",
    "gulp-uglify": "^3.0.0",
    "gulp-webserver": "^0.9.1",
    "jshint": "^2.9.5",
    "postcss-px-to-viewport": "^0.0.3",
    "postcss-write-svg": "jonathantneal/postcss-write-svg"
  }
}
```

##  gulpfile.js 配置
```javascript
/**
 * Created by mac on 2017/11/9.
 */
//导入工具包 require('node_modules里对应模块')
var gulp = require('gulp'),
    postcss = require('gulp-postcss'),
    pxtoviewport = require('postcss-px-to-viewport'),//将px单位换位vw单位
    sass = require('gulp-sass'),// CSS预处理/Sass编译
    autoprefixer = require('gulp-autoprefixer'),//添加前缀
    minifycss = require('gulp-minify-css'),//添加css压缩文件
    jshint = require('gulp-jshint'),//js代码校正
    uglify = require('gulp-uglify'),//js代码压缩
    imagemin = require('gulp-imagemin'),//图片压缩
    rename = require('gulp-rename'),//重命名
    clean = require('gulp-clean'),//清除原来文件
    concat = require('gulp-concat'),//合并文件
    notify = require('gulp-notify'),//处理报错
    cache = require('gulp-cache'),//文件对比
    livereload = require('gulp-livereload'),//监听文件发生变化，浏览器自动刷新
    webserver = require('gulp-webserver'),//开启服务器
    ghPages = require('gulp-gh-pages'),//README.md编译成html
    htmlmin = require('gulp-htmlmin'),//html压缩
    fileinclude  = require('gulp-file-include');//html模板引擎

// 样式
gulp.task('styles', function() {
    var processors = [
        pxtoviewport({
            viewportWidth: 375,
            viewportUnit: 'vw'
        })
    ];
    return gulp.src('src/styles/*.scss')
        .pipe(sass({ style: 'expanded', }))
        .pipe(autoprefixer('last 2 version', 'safari 5', 'ie 8', 'ie 9', 'opera 12.1', 'ios 6', 'android 4'))
        .pipe(gulp.dest('dist/styles'))
        .pipe(rename({ suffix: '.min' }))
        .pipe(minifycss())
        .pipe(gulp.dest('dist/styles'))
        .pipe(notify({ message: 'Styles task complete' }))
        .pipe(postcss(processors))
        .pipe(gulp.dest('dist/styles'))
        .pipe(
            postcss([
                require('postcss-write-svg')({ /* options */ })
            ])
        ).pipe(
            gulp.dest('dist/styles')
        );
});

// js
gulp.task('scripts', function() {
    return gulp.src('src/scripts/*.js')
        .pipe(jshint.reporter('default'))//检测js代码错误
        .on('error', function (err) {
            console.error('Error!', err.message); // 显示错误信息
        })
        .pipe(rename({ suffix: '.min' }))// 重命名
        .pipe(uglify())
        .pipe(gulp.dest('dist/scripts'))
        .pipe(notify({ message: 'Scripts task complete' }));
});

// 图片
gulp.task('images', function() {
    return gulp.src('src/images/*')
        .pipe(cache(imagemin({ optimizationLevel: 3, progressive: true, interlaced: true })))
        // .pipe(imagemin({
        //     progressive: true, // 无损压缩JPG图片
        //     svgoPlugins: [{removeViewBox: false}], // 不要移除svg的viewbox属性
        //     use: [pngquant()] // 深度压缩PNG
        // }))
        .pipe(gulp.dest('dist/images'))
        .pipe(notify({ message: 'Images task complete' }));
});
//html
gulp.task('html',function () {
    return gulp.src('./*.html')
        .pipe(gulp.dest('./'))
        .pipe(notify({ message: 'html task complete' }));
});
gulp.task('fileinclude', function() {
   return gulp.src('./*.html')
        .pipe(fileinclude({
            prefix: '@@',
            basepath: '@file'
        }))
        .pipe(gulp.dest('dist'));
});
// 清理
gulp.task('clean', function() {
    return gulp.src(['dist/styles', 'dist/scripts', 'dist/images'], {read: false})
        .pipe(clean());
});
//自动刷新
gulp.task('webserver',['fileinclude'],function () {
    return gulp.src( 'dist/' ) // 服务器目录（./代表根目录）
        .pipe(webserver({ // 运行gulp-webserver
            livereload: true, // 启用LiveReload
            open: true // 服务器启动时自动打开网页
        }));
});
// 预设任务及监听
gulp.task('default', ['webserver'], function() {
    gulp.run('styles', 'scripts', 'images');
    // 看守所有.scss档
    gulp.watch('src/styles/*.scss', ['styles']);

    // 看守所有.js档
    gulp.watch('src/scripts/*.js', ['scripts']);

    // 看守所有图片档
    gulp.watch('src/images/*.*', ['images']);
    //监听根目录下的所有html文件
    gulp.watch( '*.html', ['html']); // 监听根目录下所有.html文件
});



gulp.task('deploy', function() {
    return gulp.src('./dist/**/*')
        .pipe(ghPages());
});
gulp.task('testhtmlmin',function () {
    var options = {
        removeComments:true,
        collapseWhitespace:true,
        collapseBooleanAttributes:true,
        removeEmptyAttributes:true,
        removeScriptTypeAttributes:true,
        removeStyleLinkTypeAttributes:true,
        minifyJs:true,
        minifyCss:true
    }
    gulp.src('lemonhotel/*.html')
        .pipe(htmlmin(options))
        .pipe(gulp.dest('html/build'))
});


```