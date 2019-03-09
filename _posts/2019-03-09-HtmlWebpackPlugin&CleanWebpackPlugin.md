---
layout: post
title: webpack之htmlWebpackPlugin与CleanWebpackPlugin
date: 2019-03-09 
tag: webpack
---

### 使用html-webpack-plugin和clean-webpack-plugin

如果在生产环境下并没有index.html. 执行webpack是不会自动生成Index.html的。

如果希望自动生成对应的含有内用的index.html。

就需要借助插件 **webpack-html-plugin**。

**Webpack-html-plugin会在打包结束后，自动生成一个html文件，并把打包生成的js文件自动引入文件中。**

![](/images/posts/HtmlWebpackPlugin&CleanWebpackPlugin/44541dc62791f44735b39196932be26c.png)

在这之前不要忘记引入html-webpack-plugin，使用其他插件的时候也如此。

![](/images/posts/HtmlWebpackPlugin&CleanWebpackPlugin/b0e9ce0a60482553f9c56bfcf394415b.png)

**webpack的plugin的作用犹如vue的生命周期钩子，它会帮助你在webpack打包文件到某一时刻的时候做一些事情。**

当我们修改了出口文件的名称的时候，如果在这之前我们并没有将出口文件夹中的内容删除，直接打包虽然不会出现问题，但是由于打包输出的文件更新了名称，webpack会自动在出口文件夹里另外新建一个打包好的文件，之前的老文件并没有被删除，**为了能自动删除老文件，我们可以使用插件
clean-webpack-plugin（非官方插件）**。

修改出口文件名称。老文件名为bundle.js

![](/images/posts/HtmlWebpackPlugin&CleanWebpackPlugin/9a5a28fa79457037afc2f4587b5882ae.png)

打包过后，老文件依旧存在。

![](/images/posts/HtmlWebpackPlugin&CleanWebpackPlugin/ed29f398e620a2b87b1e84de5d3cb85f.png)

现在使用clean-webpack-plugin，如果想要查看关于插件的信息，可以查阅官方文档，或者是哪个插件的官方文档。

![](/images/posts/HtmlWebpackPlugin&CleanWebpackPlugin/924ec3a12c23fe8069466a5d20d7944a.png)

**注意：2019/3/6日早上起来复习的时候发现该插件已经更新，不需要再次配置参数！**

自动删除目录。

![](/images/posts/HtmlWebpackPlugin&CleanWebpackPlugin/c440d0aefa4a124367b9e96d1f95b7cb.png)

https://github.com/johnagan/clean-webpack-plugin\#options-and-defaults-optional
