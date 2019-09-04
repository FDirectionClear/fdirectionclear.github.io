---
layout: post
title: webpack环境变量的使用方法
date: 2019-09-04
tag: webpack
---

webpack环境变量的使用方法
=========================

环境变量是用方式非常简单，他可以让我们在不配置mode的情况下区分环境。

我们可以将项目中的webpack.dev.js和webpack.prod.js作出改变，去掉merge，也去掉对webpack.common.js的引入。说白了，我们是为了只是用一个配置文件就可以使用与不同的环境。直接导出配置对象即可。

就像这样：

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/4bfdfdd9a1522004eb84e369fb3eec6e.png)

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/4478f5095b52ee2de32174626f998ff4.png)

然后在webpack.common.js中引入merge。

并加入这样的代码：

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/c31c48d3a7d5c9db2bf5713b4850f7a7.png)

还没完，既然使用了env环境变量，那么就要在需要的地方传入env：

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/e2070c4352a49490fa343bcda4bed7ff.png)

这样webpack在打包的时候就会根据传入的env环境变量作出逻辑判断从而判别应该使用哪种环境下的配置文件。

当然在npm指令后传入env相关信息的写法还有很多，比如：

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/f0784517bc1bd74415e850cdcadea682.png)

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/200f3f2b3565dfa44ee7b085bf8efa0a.png)

还可以这么写，我们为env.production传递一个值：

![](/images/posts/2019-09-04-×webpack-×webpack_HuanJingBianLiang/efb4733833d798855eb384e68cc0b23a.png)

这样我们就可以根据env.production做出更细致的判断。
