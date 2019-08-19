---
layout: post
title: webpack-Shimming
date: 2019-08-19 
tag: webpack
---

Webpack Shimming的作用
======================

### 一、module的作用域留给初学者的坑

为了处理一些兼容性上的问题。比如我们有一些老的npm包，他的源代码类似于这样：

![](/images/posts/2019-08-19-webpack-Shimming/2002673c6b85748e1e28990136589027.png)

然后我们在index.js中引入这个方法：

![](/images/posts/2019-08-19-webpack-Shimming/7302f9e94cf1ebf8bb60729c1c29ba15.png)

因为每个模块的作用于都是绝对独立的，所以即使jquery已经引入在Index.js中，

在test.js中没有引入jquery，那么在test.js中\$是无法被浏览器所理解的。

顾会报错：

![](/images/posts/2019-08-19-webpack-Shimming/23dccf64d7cf39f7077b951734be949d.png)

**TIPS：模块的引入不同于简单的脚本连接。**

在test.js中引入jquery就不会发生错误。效果如下：

![](/images/posts/2019-08-19-webpack-Shimming/aac8f22fdfc25e01ade1a525ec3fe3aa.png)

### 如何不在test.js引入jquery也能正常使用\$？

可以借助webpack自带的一个插件：webpack.providePlugin();或称他为垫片（shimming）

如果我们希望脚本在发现我们有使用到’\$’这个字符串的时候**自动的在模块头部加上import
\$ from “jquery”，我们就可以这样写：**

![](/images/posts/2019-08-19-webpack-Shimming/c52c5bd5dc37e23c4dc76f21dcc0489c.png)

之后注释掉test中的jquery引入：

![](/images/posts/2019-08-19-webpack-Shimming/eb906a1561461dc0a0980c516ac30f1c.png)

重新启动打包，启动服务器，即可正常显示。

这个功能非常强大，还可以这样用：

我们希望在test.js中自动引入lodash，同时将_join代替_.join()

![](/images/posts/2019-08-19-webpack-Shimming/433a1e7b49e597c18c7ea5e92fe8f5ea.png)

我们只需要这样：

![](/images/posts/2019-08-19-webpack-Shimming/8e5f8cee258958492fd9b9cafa77beac.png)

之后就可以正常实现了。

TIP:这个过程类似于在编译过程webpack会预览一下整个脚本，如果发现webpack.ProvidePlugin下的配置，那么就会自动在整个模块的最顶端import进相关文件。这个自动的过程所带来的影响和不使用Webpack.providePlugin由我们手动配置是完全一样的。因此特别强调，自动添加的模块依旧受到代码分割规则的影响。

### webpack module下this的指向（很可能会出现bug，已经不再实用）

为了探究this的指向，我们可以直接打印this。

![](/images/posts/2019-08-19-webpack-Shimming/af632fecc92b0c729f7d8363b6690076.png)

![](/images/posts/2019-08-19-webpack-Shimming/7b3da059f25edabb3063bb6675722780.png)

可以看到，在模块中，this默认指向了模块本身，如何让所有webpack下的类似全局的this指向window呢？
我们还可以利用垫片，只不过不是使用webpack.providePlugin()了：

首先我们需要借助一个loader: imports-loader。

配置方法比较简单：

![](/images/posts/2019-08-19-webpack-Shimming/bcb75d1049435ed4fc83e2237e4cbdef.png)

首先npm下载imports-loader，之后用他率先处理.js文件，之后在给babel-loader进行转义。

按这样配置即可。

最后重新打包，就可以看到this的指向已经改向了window：

![](/images/posts/2019-08-19-webpack-Shimming/3bb4c06ea05141a46d6e78927d1e7797.png)

**但是这样似乎总是会带来bug**,其底层原理是使用IIFE将整个模块的作用域包裹，就和jquery的底层原理类似，只不过最后使用bind(window)，来将整个模块的作用域绑定到了window上。

这样当我们引入模块或者导出模块的时候，因为IIFE会不加区分的将import或者export也包裹起来，这也就造成了export和import不再最顶层作用域的bug。

![](/images/posts/2019-08-19-webpack-Shimming/fe0995d71c4e9d5a59e79ac13d6c51bc.png)

![](/images/posts/2019-08-19-webpack-Shimming/32593459dc974858fe93da01ea0c0ce1.png)

![](/images/posts/2019-08-19-webpack-Shimming/1b54f61e4350e8821df019aad83a60cd.png)
