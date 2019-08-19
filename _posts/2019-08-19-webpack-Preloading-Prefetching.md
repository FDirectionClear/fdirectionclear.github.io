---
layout: post
title: webpack之Preloading，Prefetching
date: 2019-08-19 
tag: webpack
---

webpack之Preloading，Prefetching
------------------------------------------

### Preloading

为了提高代码的录用率，我们通常会尽可能的采用异步的方式编写。但是这样又存在一个问题，比如一个登陆模态框使用异步写法，在页面加载的时候不会被加载，只有当点击登录按钮之后才会加载模态框，这样就可能造成交互反应较慢的问题，如果一开始就全部载入，又会影响浏览器整体呈现页面的时间。为了解决这个问题，我们引入preloading的概念，即在网络空闲时间加载可能需要的部分。

使用方法很简单，使用魔法注释的方式，在需要预加载的地方添加：

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/80270151288f74fc6f236debbc05fb56.png)

重新打包就会发现，我们的页面在一开始加载的时候先加载了index.html和main.js。在当main.js加载完成之后立即加载了0.js。

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/c2b76f4b6e37654d34d854e3ad0f133f.png)

然后我们点击body：

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/2b091588f779a79748a8ba1ef68b3bdb.png)

### preLoad

其实preload和prefetch的用法完全一样，只不过魔法注释的时候应该写webpackPreload:true。

它与preFetch的主要区别在于，异步引入的文件是和主文件一同加载的。这事实上并不太符合我们最佳编程的需求，我们更需要的是在网络空闲的时候加载。所以preLoad用处不如prefetch。

**TIPS：补充之前的一点可能存在于某处的错误说明：不符合代码分割的文件也会进行打包，只不过不属于任何组。**

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/81746d6252bc93c46b3e116ec573ebe9.png)

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/ad4568854df5f1eef48f711da352dc3c.png)

打包，查看情况：

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/435fcf3b56db8bd42f114226328cec58.png)

打开浏览器，点击body：

![](/images/posts/2019-08-19-webpack-Preloading-Prefetching/6c843ccdeb78d443fd761fa4f9b238f6.png)

发现0.js就是异步引入的lodash。
