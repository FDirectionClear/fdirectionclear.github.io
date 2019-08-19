---
layout: post
title: Webpack相关热模块更新（HMR）1
date: 2019-08-19 
tag: webpack
---

Webpack相关热模块更新（HMR）1
=============================

Webpack-dev-server虽然能在本地起一个服务器，还能帮助我们自动打开浏览器并刷新页面，但终究还是得刷新，及时这个过程是自动的。如果我们编写的是一些动态代码，比如需要触发事件添加样式。那么每一次刷新就会使之前触发事件后的样式消失，需要重新触发样式。

现在我们有更方便的方式，浏览器连刷新都不刷新，直接在现有页面的基础上修改页面渲染。

听上去是在是在炫酷了，让我们来看看如何利用webpack来实现：

### 一、Webpack热模块更新——HMR

HMR的开启比较简单，引用并使用一个webpack自带的插件，并且为devServer增加属性hot属性，配置成true即可。

![](/images/posts/2019-08-19-webpack-hmr1/90426eb9f8f41957c2876d7e4d7b29dc.png)

配置hot与hotOnly之后引入webpack模块。就是为了使用webpack自带的Webpack.HotModuleReplacementPlugin()插件。

![](/images/posts/2019-08-19-webpack-hmr1/3c9758ce153fcd36dfb6681519079733.png)

![](/images/posts/2019-08-19-webpack-hmr1/9ef7cc66ab867373281f201582d50b85.png)

之后重启webpack-dev-server，只需要再次运行npm run
start，自动打开浏览器就是HMR模式了。

**注意：每次修改webpack配置文件都需要重新打包！**

*TIPS:*

1.  *HMR并不是webpack默认开启的功能，需要我们手动配置。*

2.  *HMR是不刷新浏览器的。这也就意味着页面会直接得到修改。但是在不额外写热更新相关代码的情况下，HMR只支持样式文件，不适用于js等文件。*

*3、在.vue工程中，HMR已经对js文件得到了支持。*

### 二、实验例子

![](/images/posts/2019-08-19-webpack-hmr1/4c19cda352291331e8e368d43c34daef.png)

Css：

![](/images/posts/2019-08-19-webpack-hmr1/f1d209f60f8b72fc3ee32c5a85d51ab6.png)

![](/images/posts/2019-08-19-webpack-hmr1/8bfc57d58fa44aa4013a07dfb83e7e4a.png)

直接修改代码，改变css，修改背景颜色。

![](/images/posts/2019-08-19-webpack-hmr1/9db6fa54fea3b2fd978493d983068fb1.png)

注意不刷新浏览器！ 不需要对浏览器做出任何操作！

![](/images/posts/2019-08-19-webpack-hmr1/760909e414d1ecdf618737c42c0db0df.png)

可以发现背景颜色自动的改变了，而且浏览器没有执行任何刷新操作！这实在是太炫酷了。
