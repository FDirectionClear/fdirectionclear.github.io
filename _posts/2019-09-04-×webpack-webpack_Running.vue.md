---
layout: post
title: 如何在webpack中编译.vue文件
date: 2019-09-04
tag: webpack
---

如何在webpack中编译.vue文件
===========================

如果我们希望书写的是Jsx文件，那么我们就需要为babel安装一些插件，然后使用babel来便以这些Jsx文件。

### 使用babel-loader编译jsx

为了能让babel-loader可以编译jsx，我们需要引入一个核心插件

babel-plugin-transform-vue-jsx

cnpm下载并安装它：

![](/images/posts/2019-09-04-×webpack-webpack_Running.vue/6a98692d86165a086b0a3b8e546ae957.png)

然后修改.babelrc来引入这个插件。

![](/images/posts/2019-09-04-×webpack-webpack_Running.vue/842e3619e026c2c167fda67031c63b1a.png)

为webpack.config.js增加rules;以便让webpack可以打包编译.jsx文件

![](/images/posts/2019-09-04-×webpack-webpack_Running.vue/ce5da0e79d624c65ac8582e232118f7e.png)

然后我们可能还需要一些依赖文件，只需要根据报错的提醒安装文件即可。

（不同版本的额依赖文件可能不同，根据报错提醒安装相关的peer是一个好的方法）

![](/images/posts/2019-09-04-×webpack-webpack_Running.vue/eb58a209826f4aed1c798454767ecc4e.png)

然后下载安装这个插件

![](/images/posts/2019-09-04-×webpack-webpack_Running.vue/8a8e3d992b8a897f827026d56084e3a3.png)

之后便可以成功编译打包了。
