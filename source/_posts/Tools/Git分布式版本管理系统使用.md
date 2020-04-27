---
title: Git分布式版本管理系统使用
date: 2018-03-09 16:51:16
tags: "git"
categories: "Tools"
---
#快速开始

## 安装

### windows安装
在Windows上使用Git，可以从Git官网直接[下载安装程序](https://gitforwindows.org/)，（网速慢的同学请移步国内镜像），然后按默认选项安装即可。

安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！
安装完成后，还需要最后一步设置，在命令行输入：

```javascript
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```

### mac安装
如果你正在使用Mac做开发，有两种安装Git的方法。

一是安装homebrew，然后通过homebrew安装Git，具体方法请参考homebrew的文档：http://brew.sh/。

第二种方法更简单，也是推荐的方法，就是直接从AppStore安装Xcode，Xcode集成了Git，不过默认没有安装，你需要运行Xcode，选择菜单“Xcode”->“Preferences”，在弹出窗口中找到“Downloads”，选择“Command Line Tools”，点“Install”就可以完成安装了。

<!-- more -->

## 新建本地仓库

使用Git前，需要先建立一个仓库(repository)。您可以使用一个已经存在的目录作为Git仓库或创建一个空目录。

使用您当前目录作为Git仓库，我们只需使它初始化。

```javascript
git init  XXX
```

## 添加新文件

我们有一个仓库，但什么也没有，可以使用add命令添加文件。

```javascript
git add filename
```
可以使用add... 继续添加任务文件。

## 提交版本
现在我们已经添加了这些文件，我们希望它们能够真正被保存在Git仓库。

为此，我们将它们提交到仓库。

```javascript
git commit -m "Adding files"
```

如果您不使用-m，会出现编辑器来让你写自己的注释信息。

当我们修改了很多文件，而不想每一个都add，想commit自动来提交本地修改，我们可以使用-a标识。

```javascript
git commit -a -m "Changed some files"
```

git commit 命令的-a选项可将**所有被修改或者已删除的且已经被git管理的文档**提交到仓库中。

<font color = "red">千万注意，-a不会造成新文件被提交，只能修改</font>

## 发布版本
我们先从服务器克隆一个库并上传。

```javascript
git clone ssh://example.com/~/www/project.git
```

现在我们修改之后可以进行推送到服务器。

```javascript
git push ssh://example.com/~/www/project.git
```

## 取回更新
如果您已经按上面的进行push，下面命令表示，当前分支自动与唯一一个追踪分支进行合并。

```javascript
git pull
```

从非默认位置更新到指定的url。

```javascript
git pull http://git.example.com/project.git
```

## 删除
如何你想从资源库中删除文件，我们使用rm。

```javascript
git rm file
```

## 分布与合并
分支在本地完成，速度快。要创建一个新的分支，我们使用branch命令。

```javascript
git branch test
```

branch命令不会将我们带入分支，只是创建一个新分支。所以我们使用checkout命令来更改分支。

```javascript
git checkout test
```

第一个分支，或主分支，被称为"master"。

```javascript
git checkout master
```

对其他分支的更改不会反映在主分支上。如果想将更改提交到主分支，则需切换回master分支，然后使用合并。

```javascript
git checkout master
git merge test
```

如果您想删除分支，我们使用-d标识。
```javascript
git branch -d test
```