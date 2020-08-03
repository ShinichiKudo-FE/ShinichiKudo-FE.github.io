---
title: 如何规范你的git提交?
tags: 'git'
categories: 'Tools'
date: 2020-08-03 13:33:53
---

[原文地址](https://www.jianshu.com/p/5522df15c9ec)

# 一、为什么需要规范？

无规矩不成方圆，编程也一样。

如果你有一个项目，从始至终都是自己写，那么你想怎么写都可以，没有人可以干预你。可是如果在团队协作中，大家都张扬个性，那么代码将会是一团糟，好好的项目就被糟践了。不管是开发还是日后维护，都将是灾难。

这时候，有人提出了何不统一标准，大家都按照这个标准来。于是 ESLint，JSHint 等代码工具如雨后春笋般涌现，成为了项目构建的必备良品。

`Git Commit` 规范可能并没有那么夸张，但如果你在版本回退的时候看到一大段糟心的 Commit，恐怕会懊恼不已吧。所以，严格遵守规范，利人利己。

二、具体规则

先来看看公式：

**`<type>(<scope>): <subject>`**

> type 用于说明 commit 的类别，只允许使用下面7个标识。
* feat：新功能(feature)
* fix：修补
* bugdocs：文档(documentation)
* style：格式(不影响代码运行的变动);
* refactor：重构(即不是新增功能，也不是修改bug的代码变动)
* test：增加测试
* chore：构建过程或辅助工具的变动

> scope 用于说明 commit 影响的范围，比如`数据层、控制层、视图层`等等，视项目不同而不同。

> subject 是 commit 目的的简短描述，不超过50个字符。

* 以动词开头，使用第一人称现在时，比如change，而不是changed或changes
* 第一个字母小写
* 结尾不加句号(.)

# 三、异常处理

我们先来看看这个异常提醒：

```js
INVALID COMMIT MSG: does not match "<type>(<scope>): <subject>" ! 
jartto:fix bug 
```

这里之所以报出这个警告，是因为我的提交出现了两个问题：

* 其一，使用了规范外的关键字;
* 其二，很细节的问题，`jartto：`后少了空格;

这时候我才回忆起来，当时提交一直失败，情急之下直接强制提交，所以以后的提交都会抱出这个异常。大致意思就是：

> 你的之前的 Commit 不合格～你的之前的 Commit 不合格～你的之前的 Commit 不合格

这时候就很烦了，我们只能去将之前的错误修正，那么如何操作呢?

# 四、如何修改之前的 commit 信息？

其实并不复杂，我们只需要这样做:

* 1、将当前分支无关的工作状态进行暂存 `git stash`
* 2、将 HEAD 移动到需要修改的 commit 上 `git rebase 9633cf0919^ --interactive`
* 3、找到需要修改的 commit ,将首行的 pick 改成 edit
* 4、开始着手解决你的 bug
* 5、`git add `将改动文件添加到暂存
* 6、`git commit –amend` 追加改动到提交
* 7、`git rebase –continue` 移动 HEAD 回最新的 commit
* 8、恢复之前的工作状态 `git stash pop`

大功告成，是不是想把整个 Commit 都修改一遍
  
# 五、项目中使用

这时候问题又来了，为什么我提交的时候会有警告，这个又是如何做到的呢?

这时候，我们需要一款 Node 插件 `validate-commit-msg` 来检查项目中 `Commit message` 是否规范。

* 1.首先，安装插件：

```js
npm install --save-dev validate-commit-msg 
```

* 2.使用方式一，建立 .vcmrc 文件：
```js
{ 
  "types": ["feat", "fix", "docs", "style", "refactor", "perf", "test", "build", "ci", "chore", "revert"], 
  "scope": { 
    "required": false, 
    "allowed": ["*"], 
    "validate": false, 
    "multiple": false 
  }, 
  "warnOnFail": false, 
  "maxSubjectLength": 100, 
  "subjectPattern": ".+", 
  "subjectPatternErrorMsg": "subject does not match subject pattern!", 
  "helpMessage": "", 
  "autoFix": false 
}
```

* 3.使用方式二：写入 package.json

```json
{ 
  "config": { 
    "validate-commit-msg": { 
      /* your config here */ 
    } 
  } 
}
```

* 4.可是我们如果想自动使用 ghooks 钩子函数呢？

```js
{ 
  … 
  "config": { 
    "ghooks": { 
      "pre-commit": "gulp lint", 
      "commit-msg": "validate-commit-msg", 
      "pre-push": "make test", 
      "post-merge": "npm install", 
      "post-rewrite": "npm install", 
      … 
    } 
  } 
  … 
} 
```

在 ghooks 中我们可以做很多事情，当然不只是 `validate-commit-msg`哦。

# 六、Commit 规范的作用

* 提供更多的信息，方便排查与回退;
* 过滤关键字，迅速定位;
* 方便生成文档;

# 七、生成 Change log

正如上文提到的生成文档，如果我们的提交都按照规范的话，那就很简单了。生成的文档包括以下三个部分：

**New features**

**Bug fixes**

**Breaking changes**

每个部分都会罗列相关的 `commit` ，并且有指向这些 `commit` 的链接。当然，生成的文档允许手动修改，所以发布前，你还可以添加其他内容。

这里需要使用工具 `Conventional Changelog` 生成 `Change log`：

```js
npm install -g conventional-changelog 
cd jartto-domo 
conventional-changelog -p angular -i CHANGELOG.md -w
```

为了方便使用，可以将其写入 `package.json` 的 scripts 字段。

```js
{ 
  "scripts": { 
    "changelog": "conventional-changelog -p angular -i CHANGELOG.md -w -r 0" 
  } 
}
```

这样，使用起来就很简单了：

```js
 npm run changelog
```

# 八、总结

看完文章，你还会如此放荡不羁吗?你还会随心所欲的编写 Commit 吗?你还会如此 git commit -m "hello world"提交吗?

答案是否定的，因为使用了钩子函数，你没有机会了，否则将是无穷无尽的恢复 Commit。这倒可以养成良好的提交习惯。


