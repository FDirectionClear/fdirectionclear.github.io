---
layout: post
title: webpack-dev-server
date: 2019-09-04
tag: webpack
---

### 一、webpack watch自动观测打包

之前的开发效率很低下，因为我们每一次修改代码都需要npm run
bundle。如果我们每一次的小改动都要经过npm run
bundle，然后重新刷新页面，是一件非常麻烦的事。幸好webpack
为我们提供了watch指令来让webpack自动观测到文件的变化，然后自动打包。

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/9c9424d16b074e5232e5c96c6de6f72b.png)

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/f19ee963653567497663d3ee2832d1ce.png)

虽然自动打包看上去很方便，但是我们的项目还是有不便之处，比如没有服务器（之前一直用的wampserver）的配合，不能自动打开页面，不能自动刷新页面等等。

### 二、webpack-dev-server

如果需要一个能自动观察代码变化随之打包，之后还能帮助搭建一个本地服务器，另外自动打开浏览器预览的炫酷效果，就需要借助webpack-dev-server这个插件了。

**接下来记录下安装的关键几步：**

1.  **先安装webpack-dev-server**

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/2879deadef1b322abaa62240085333b7.png)

为了方便启动webpack-dev-server，可以配置npm快捷脚本：

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/e29c39d23238028125786a5c6aeabb49.png)

还没完，我们还需要在webpack.config.js中配置webpack-dev-server.

**2、配置devServer**

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/361aefb97ebadc389eaffc95ad28c39b.png)

一般情况下，我们的服务器都是建设在生产环境下的。这个很好理解，因为我们打开浏览器看到的index.html和他们引入的静态资源等都是从生产环境中打包过后的文件。因此，我们只需要将服务器架设在生产环境目录下即可。

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/70a7e471ec4cd1499ddf1a8ef1470b8e.png)

执行npm run
start之后，就会自动在localhost:8080端口创建了一个本地node服务器，当然端口号(host)也是可以设置的，devServer还有相当多的可配置内容，比如before(),
historyFallback...,详情见官网。

现在我们的webpack已经具备webpack
--watch的功能。而且在有代码更改的时候，我们不需要手动刷新浏览器，devServer就会为我们自动刷新浏览器。我们在写代码的时候，只要入口文件以及他依赖的文件进行了保存操作，就会自动进行刷新页面的操作。但是现阶段并不能自动打开浏览器同时访问Localhost8080;

如果需要浏览器自动打开并访问8080，就需要继续为devServer进行配置，增加open属性，并设置为true；这真滴是极其炫酷。

（补充图片）：

![](/images/posts/2019-09-04-×webpack-webpack-dev-server/a907cc483236628cd91e1a275d0800a2.png)

**开启服务器有什么好处呢？**

**这里只着重谈论关键的两点：**

1.  可以不借助wampserver开启一个node本地服务器。

2.  网页是通过http协议请求的，因此可以进行ajax传输。如果只进行npm run
    bundle，那么就是单纯的打开了这个文件，只是通过file请求，是不允许发送ajax请求的。（借助wampserver也是这个道理，也是通过http请求）。
