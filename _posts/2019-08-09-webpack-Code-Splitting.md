---
layout: post
title: html-webpack-plugin和clean-webpack-plugin的使用和入坑小记
date: 2019-06-15 
tag: webpack
---

### 为什么要代码分割？

代码分割并不是webpack独有的概念，在webpack诞生以前就存在了代码分割的思想和做法。

代码分割的好处：能够把静态资源分割出去，比如类似vue、和lodash之类的静态类库，我们就可以把他们分割出去，形成单独的文件。如果不进行分割，这类大的静态资源就会和我们的业务逻辑打包到同一个js文件中,业务逻辑代码一旦变化，如果用户重新加载页面，浏览器就不得不把这个巨大的js文件再次加载一次。

如果能够分割代码，那么当我们的业务逻辑代码发生变化的时候，浏览器只要加载我们变动的js文件，那些静态资源就会从缓存中读取。节省了加载的速度。

另外，并行加载的速度似乎更快。打包到一个js中，就只有一个script引用这个js文件，如果分割出好几个js文件，就会有多个script引用多个js文件。就会产生并行加载。

一般的代码分割如何实现？
------------------------

一个思路是将要分割的静态代码绑定到全局对象上（针对需要导出模块的文件，不需要导出模块的，直接写在另一个文件中应该即可）。然后webpack打包，设置多个入口文件，那些静态的代码就可以通过多个入口打包成多个单独的js文件。然后HtmlPlugin就会自动的在index.js文件中按顺序引入文件。

因为代码分成了多个文件，因此，当有代码发生变化的时候，只需要重新加载对应的js即可。

**举个例子：**

lodash是一个非常实用的工具库。他就是一个典型的静态资源。我们要对他进行分割，其实非常简单。

先来看看不进行代码分割。

![](
/images/posts/2019-08-09-webpack-Code-Splitting/7c086e942b7e2734459c6c008d56a5c9.png)

这样不进行代码分割的情况下，
lodash文件就会和我们的业务逻辑同一打包到同一个js文件下。当我们的业务逻辑也很大的时候，性能就会极差，发生业务变更，用户就不得不将业务逻辑和lodash这样的静态资源一同打包。

如果我们能把那些静态资源写进一个js文件中，然后让webpack把它打包成一个模块，这样就可以使静态资源的js和我们业务逻辑的js分离开了。

但是这样又出现了一个问题，我们把虽把静态模块和业务模块分隔开，但是当我们在业务模块中调用静态模块的导出内容时又会经过import
xxx from
‘静态资源’的情况，webpack就又会把静态资源和业务逻辑打包到同一个包中，这样我们分割出的静态模块就毫无意义。

所以我们为了避免使用import，可以在静态文件中把需要导出的模块绑定到Window对象下，这样，只需要我们在index.html中按顺序引用js文件，即使不import，也可以在任何一个模块中共享静态模块中导出的内容了。

分隔代码我们可以这样做：

![](
/images/posts/2019-08-09-webpack-Code-Splitting/cea295b17b7b9d84ff86abeca9bc1296.png)

![](
/images/posts/2019-08-09-webpack-Code-Splitting/3fdc6f2b7beb37bd8d6469afa017a04e.png)

![](
/images/posts/2019-08-09-webpack-Code-Splitting/97dababcc0758f3859ba6163b62c76ae.png)

这个过程就是一个代码分割的过程。当我们修改index.js时候，浏览器只需要重新加载index.js即可，对于打包好的lodash只需要从缓存中读取即可。

如何通过webpack进行更高级的代码分割？
-------------------------------------

上面这种方式式手动实现的代码分割，在使用webpack时，我们可以结合webpack4自身绑定的splitChunks插件来实现自动代码分割。

我们只需要增加一个optimization选项——splitChunks。开启这个选项后，我们就不必手动分离静态文件，正常import静态文件导出的内容即可，webpack已经能智能的根据import的内容自动实现类似上部分所讲的代码分割。

![](
/images/posts/2019-08-09-webpack-Code-Splitting/ab4b1764a2a67712a6d6e1caa95ebbd6.png)

![](
/images/posts/2019-08-09-webpack-Code-Splitting/62cf6e99d191302ae609f81d992a73f7.png)

重新打包，我们就可以看到在bundle目录下多了一个lodash打包来的文件。

![](
/images/posts/2019-08-09-webpack-Code-Splitting/4a4b6cfa33ca5f302fb5362ebb36d9f9.png)

这个额外的vendors就是来自main.js的分割文件。换句话说，他就是lodash.js。

如何动态实现代码分割？
----------------------

所谓动态实现代码分割就是异步的import文件。

看下面的例子：

![](
/images/posts/2019-08-09-webpack-Code-Splitting/9ff287aa410e17e666d3ed415f608764.png)

*TIP：*

*这种异步加载文件还处于实验阶段，需要babel插件去进行转义，*

这个插件的名字很长，\@babel/plugin-syntax-dynamic-import

下载完成后

然后在.babelrc文件中配置这个插件：

![](
/images/posts/2019-08-09-webpack-Code-Splitting/d51d0efb01e48d2e24d32965b955922c.png)

配置好.babelrc文件后就可以重新打包，**值得注意的是，这种异步动态打包文件的方式不需要配置optimization.splitChunks，webpack会自动的对这些文件进行代码分割。**

**但是更需要注意的是，虽然实现代码分割不需要配置，但是不代表就一定不必配置，因为还有很多时候，比如要设置分割出来的文件的名称的时候，就需要配置optimization了。**

**这也就是说，无论是同步的代码分割还是异步的代码分割，配置optimization都是有意义的。**

重新打包后，结果如下：

![](
/images/posts/2019-08-09-webpack-Code-Splitting/32db44503c7e6b37cb6142219d97908b.png)

在这里引出一个问题：生成的js的文件名为0.js，那该如何限制生成的js的文件名呢？

为了解决这个问题，我们可以使用**魔法注释**的方式小修改一下代码：

![](
/images/posts/2019-08-09-webpack-Code-Splitting/e75cd58aedd23cd8feaf3a82d03359e0.png)

然后重新打包就会发现名字已经发生了改变：

![](
/images/posts/2019-08-09-webpack-Code-Splitting/ccdefc1c35a32bfac1d4e3d4732b008d.png)

但是发现名字前还由一个vendors\~，这个该如何修改呢？、

这就需要通过配置optimization.splitChunks了。

关于optimization的配置内容我会在下文中记录。
