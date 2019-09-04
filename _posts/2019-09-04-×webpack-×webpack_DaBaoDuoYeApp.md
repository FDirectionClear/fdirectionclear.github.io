---
layout: post
title: Webpack多页面打包配置
date: 2019-09-04
tag: webpack
---

Webpack多页面打包配置
=====================

虽然现在大多数webpack的应用场景都是单页面应用（SPA），但是也有一些时候会需要多页面打包的需求。也就是说，SPA项目中只有一个index.html，而在多页面里就会有很多html文件。

那该如何使用webpack生成多个html文件？
-------------------------------------

操作很简单，我们需要几个页面，就在plugins里配置html-webpack-plugin几次即可，别忘记因为不同的页面可能需要不同的js文件，因此，entry的配置也需要改变：

以下场景为例，加入我们需要生成一个index.html 和
list.html，然后各自引入一些公用的js文件和独特的js文件，比如index.js需要引入main.js而不需要引入list.js，而list.js反之：

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaoDuoYeApp/67d0bb9b4c5cb17f85e7c9c1ba2b0a01.png)

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaoDuoYeApp/034054d1ca4b655b288cecd6ac216b9e.png)

你会发现在htmlwebpackplugin中多出了两个配置filename和chunks，filename没什么可说的，因为会自动生成多个html文件，所以需要保证文件名不能相同，另外为了让html区意识到不同的html文件需要引入独特的js，我们就需要增加chunks属性，这个属性是一个数组，里面的写一些这个html需要引入的js文件名。

然后执行打包，生成目录如下：

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaoDuoYeApp/5403150d3542b0dfb58c215991d52ed8.png)

进入list.html中，ctrl +
f搜索main,会发现并不存在，说明在list.js中并没有引入main.js文件。

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaoDuoYeApp/63933bd85b4f63bb78d234b5f5af2661.png)
