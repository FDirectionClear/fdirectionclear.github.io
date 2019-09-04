---
layout: post
title: Webpack如何打包多个文件
date: 2019-09-04
tag: webpack
---

Webpack如何打包多个文件
=======================

如何打包多个文件？

先来看看多次打包同一个文件，这样就会受到启发。

如果不指定output中的filename。那么entry打包出的文件名就默认使用对应的出口键名。

这里的就是main和sub。entry如果只设置一个路径，不指定键名，那么默认就是main。

![](/images/posts/2019-09-04-×webpack-webpack_webpackDuoWenJianDaBao/7498d90b4acb468e5cf0ef357e82a8ec.png)

正确的操作应该是这样

![](/images/posts/2019-09-04-×webpack-webpack_webpackDuoWenJianDaBao/b3eb4fde48c7229027a80083961c4dfc.png)

有些时候，我们可能把打包完成的js文件上传到一个cdn中，想要利用cdn中的静态资源。这时候打包后的html文件的script的src如果还是直接引用当前目录下的内容肯定不靠谱。（打包好的js上传到了cdn上，人家要cdn上js的文件，你自动引入当前目录下的js文件地址肯定不行）。

我们这时就需要为src的文件名前加入cdn的网址.如何自动加入，就要使用output选项的publicPath选项。

不设置publicPath

![](/images/posts/2019-09-04-×webpack-webpack_webpackDuoWenJianDaBao/d2540aa363a2f226038c1fae9f9e862c.png)

设置publicPath

![](/images/posts/2019-09-04-×webpack-webpack_webpackDuoWenJianDaBao/7c82fc691257a99924966b32119828cf.png)

![](/images/posts/2019-09-04-×webpack-webpack_webpackDuoWenJianDaBao/fe25141828bd63a79e8ac0801dbfabf7.png)

如果cdn索引上没文件，控制台报错就理所当然了。

![](/images/posts/2019-09-04-×webpack-webpack_webpackDuoWenJianDaBao/5838604951408c024360d7ad57b6f9cd.png)
