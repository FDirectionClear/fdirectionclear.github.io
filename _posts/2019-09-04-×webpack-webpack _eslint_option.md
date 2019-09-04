---
layout: post
title: 在webpack中配置eslint
date: 2019-09-04
tag: webpack
---

在webpack中配置eslint
=====================

再开发中，有很多人的编辑器不能使用vscode中的eslint插件，也就不能明显的在编辑器中根据标记发现不规范的错误。他们只能通过命令行
npx eslint src来查看错误，这是非让人恶心的一件事。

虽然我们不能让没有eslint插件的编辑器显现错误，但是我们可以让eslint结合webpackDevServer来实现页面上的动态提示。

**第一步：**

下载eslint-loader

**第二步**：

将所有的.js文件打包前先使用eslint-loader来检测错误，此时如果存在错误，会在打包时在命令行中有所提示。需要注意eslint-loader需要在babel-loader之前执行。

需要这样写：

![](/images/posts/2019-09-04-×webpack-webpack _eslint_option/8f2bd2e0ac37291f600a85b003e1004d.png)

但是这样的报错还是不够直观，错误信息会在打包的时候显示在命令行，这和npx eslint
src没啥大区别。

**第三步：**

这是我们可以借助devServer提供的overlay:true。

开启这个选项后，我们可以发现如果存在错误，错误会遮挡在浏览器的页面上。这个错误提示是随devServer一起热更新的，修复好一项就会在浏览器的页面中少去一项的显示。一旦修复完所有错误，遮罩层会自动消失。

下图是浏览器页面上显示报错信息：

![](/images/posts/2019-09-04-×webpack-webpack _eslint_option/48286f4080550a993983b00e5e2a0577.png)

但是大多情况下我们都不会选择在webpack的配置文件中取配置eslint-loader，因为这会降低打包速度。我们通常把配置写在单独的配置文件.eslintric中，就和.babelrc一样的性质。
