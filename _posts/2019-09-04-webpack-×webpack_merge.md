---
layout: post
title: Webpack-merge混合配置文件
date: 2019-09-04
tag: webpack
---

Webpack-merge混合配置文件
=========================

### 一、dev配置和build配置以及common配置分离

开发环境和生产环境的切换需要不断地修改webpack.config.js。

为了方便，我们创建两个版本的配置文件，分别是生产环境下的配置文件和开发环境下的配置文件：webpack.dev.js
webpack.prod.js。

![](/images/posts/2019-09-04-webpack-×webpack_merge/19218cf60921d1d61ba44dbe8df7f298.png)

在生产环境下可以取消或者更改很多开发环境下的配置。

比如HRM、sourceMap...

然后为了方便的应用配置文件，配置npm快捷脚本:

![](/images/posts/2019-09-04-webpack-×webpack_merge/11f7a818c532eaa926d576c26529ac46.png)

我们发现，在webpack的两个环境的配置文件中依旧存在很多代码相同的部分，这时候我们可以在创建一个webpack配置文件webpack.common.js。然后将共同的部分写在这个文件之中。

![](/images/posts/2019-09-04-webpack-×webpack_merge/23c4a75d0681cdab7704d7d2f2d3b2fb.png)

![](/images/posts/2019-09-04-webpack-×webpack_merge/1bcb5db532f1f0b9ca327ff03011cd31.png)

![](/images/posts/2019-09-04-webpack-×webpack_merge/5bfa2b6611d0030545a3fde25768145f.png)

### 二、webpack.common.js混合

如何将三个配置文件链接起来呢？

需要一个webpack的插件webpack-merge （merge的汉语意思是融入）

下载前先安装

![](/images/posts/2019-09-04-webpack-×webpack_merge/d184f7017b7bbe3d2403dd7759860ca3.png)

（回顾一下ES MODULE和CommonJS的模块导入导出写法：

https://www.cnblogs.com/lxg0/p/7774094.html

）针对模块引入，我的想法是当我们在一个文件里使用到其他文件模块导出的内容时，那么js就会进入那个模块的所属文件的执行空间进行代码的执行。执行完毕后带着结果回来继续执行本文件的代码。

下面是我的实验：

![](/images/posts/2019-09-04-webpack-×webpack_merge/f757000c350c4bb12ebf0fb299720236.png)

![](/images/posts/2019-09-04-webpack-×webpack_merge/f130af3c4442d216bbed04ee5e8e0287.png)

最后的结果，验证了我的想法是正确的。

![](/images/posts/2019-09-04-webpack-×webpack_merge/9be787efefafd7b71c836b1eb702d214.png)

*PS：这种情况是ES模块打出方法，如果用AMD的module.exports导出方法可能效果就完全不同了。*

**言归正传，回到webpack的配置文件混合方法：**

然后进行下面的操作：

![](/images/posts/2019-09-04-webpack-×webpack_merge/17d2f13b631ef2ee3b5dceb38f01b574.png)

![](/images/posts/2019-09-04-webpack-×webpack_merge/dc92350891fc1043894456b62f7e2552.png)

然后混合连接完成，就可以进行打包了。

有些时候我们需要将配置文件移动到其他文件中进行整理：

这时候因为配置文件的位置进行了改变，所以需要修改配置文件的引用路径。

比如文件目录为：

![](/images/posts/2019-09-04-webpack-×webpack_merge/e08607f262c408448118a97b5592df42.png)

在package.json文件中重新定义npm脚本：

![](/images/posts/2019-09-04-webpack-×webpack_merge/17050016cbc87090150f9d1881576051.png)

然后就可以正常的执行了。
