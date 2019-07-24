---
layout: post
title: html-webpack-plugin和clean-webpack-plugin的使用和入坑小记
date: 2019-06-15 
tag: webpack
---

### 一、html-webpack-plugin

如果在生产环境下并没有index.html. 执行webpack是不会自动生成Index.html的。

如果希望自动生成对应的且含有静态资源引用的index.html就需要借助插件
**html-webpack-plugin**。他是一个官方插件，老版本中很多人喜欢称呼他为html-plugin，其实说的就是他。我们可以在webpack的官方网站中找到他。

**html-webpack-plugin会在打包结束后，自动生成一个html文件，并把打包生成的js文件自动引入文件中（当然这里也不局限于js文件，对于分割出来的css文件也会自动引入）。**

![](/images/posts/2019-06-15-webpack-html&Clean-webpack-plugin/44541dc62791f44735b39196932be26c.png)

在这之前不要忘记引入html-webpack-plugin，使用其他插件的时候也如此。

![](/images/posts/2019-06-15-webpack-html&Clean-webpack-plugin/b0e9ce0a60482553f9c56bfcf394415b.png)

*TIP:
webpack的plugin的作用犹如vue的生命周期钩子，它会帮助你在webpack打包文件到某一时刻的时候做一些事情。htmlWebpackPlugin是在打包后执行，而CleanWebpackPlugin是在打包前执行*

### 二、Clean-Webpack-Plugin

当我们修改了出口文件的名称的时候，如果在这之前我们并没有将出口文件夹中的内容删除，直接打包虽然不会出现问题，但是由于打包输出的文件更新了名称，webpack会自动在出口文件夹里另外新建一个打包好的文件，之前的老文件并没有被删除，**为了能自动删除老文件，我们可以使用插件
clean-webpack-plugin （非官方插件）**。

修改出口文件名称。老文件名为bundle.js

![](/images/posts/2019-06-15-webpack-html&Clean-webpack-plugin/9a5a28fa79457037afc2f4587b5882ae.png)

打包过后，老文件依旧存在。

![](/images/posts/2019-06-15-webpack-html&Clean-webpack-plugin/ed29f398e620a2b87b1e84de5d3cb85f.png)

现在使用clean-webpack-plugin，如果想要查看关于插件的信息，可以查阅官方文档，或者是哪个插件的官方文档。

**注意：2019/3/6日早上起来复习的时候发现该插件已经更新，不需要再次配置参数！**

### 三、车祸现场

*TIP：多次坑点！html-webpack-plugin的升级比较频繁，经常改变使用方式，早些版本需要向CleanWebpackPlugin中传入一个数组参数，表示要清除的目录。但是现在不需要了，他自动清除output.path目录下的文件。当然，也支持传入对象的方式，更细致的自定义清除功能。*

*入坑现场：*

![](/images/posts/2019-06-15-webpack-html&Clean-webpack-plugin/c440d0aefa4a124367b9e96d1f95b7cb.png)

详细配置查他的github仓库：

https://github.com/johnagan/clean-webpack-plugin\#options-and-defaults-optional

*另外，当前的版本的（6月15日）clean-webpack-plugin的构造函数需要结构，源码中不再是export
default的形式导出CleanWebpackPlugin模块了！所以引入时需要解构。*

![](/images/posts/2019-06-15-webpack-html&Clean-webpack-plugin/54889e0d5cdefbe3a57300c87a70cfbb.png)
