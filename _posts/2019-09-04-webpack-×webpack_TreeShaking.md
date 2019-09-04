---
layout: post
title: Webpack TreeShaking
date: 2019-09-04
tag: webpack
---

Webpack TreeShaking
===================

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/ad953808745f044dddb55baad65f0cc1.png)

之前谈到的/babelrc的配置中，在”presets”属性下配置了一个转码规则\@babel/preset-env（如果还需要其他的转码规则，就需要安装其他的转码规则，并在这里进行配置，比如说”\@babel/preset-react”），给这个转码规则又配置了一个属性”useBuiltIns”:”usage”。表示我们只需要对文件中用到的ES6对象和方法进行补充即可，不需要将所有的对象和方法都不管用没用得到全都打包进去，这无疑会使我们的html文件变得很大。

另外，使用了useBuiltIns:usage之后，我们就不用再入口文件的头部显示的引入\@babel/polyfill了。

如果继续引入\@babel/polyfill会作出警告。

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/adfef38af819eebc8c3037b32acdf9c9.png)

### 一、Tree Shaking概念

比如这样一个场景，我们在入口文件Index.js引入一个js文件main.js。

在main.js下我们定义并导出两个函数：

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/91a42fdf96c4ce6b90d593f011e4e752.png)

但是在index.js中我们只引入add模块。

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/10c06a14e180262597847b537e851473.png)

进行打包，并没有出现任何问题，在控制台正常输出了3。

但是一个细节的问题产生了，我们只是用add模块，webpack在打包的时候却并没有只引入了add模块，就连我们未引入的minus模块也一起打包了。

查看出口文件下的js文件，我们可以看到minus也导入了进去。

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/89dddd795a3da6fc76b56be9d76d97c5.png)

如果我们详解解决这个问题，那我们就可以使用Tree Shaking的概念。

就比如我们所引入的所有js文件导出的模块形成一颗大树（如果没有导出，treeShaking默认忽略了没有任何模块导出的文件，而我们需要我们用不着的模块摇晃掉。这就是Tree
Shaking的概念。

**另外TreeShaking只能使用ES6 MODULE的引入方式。**

也就是这样的形式：

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/6feda7e5bbf7fcdb4d75d32ade8465db.png)

在mode:”development”下，treeShaking默认是没有打开的。

我们需要如下配置：

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/ae7c18a1de64ba1de77a450816c0941f.png)

然后还没完，我们还需要在package.json增加一些配置：

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/09ff38c7c29a7ec3886a5d0922580773.png)

False是默认值，**sideEffect的意思是，我们在进行打包的时候让TreeShaking正常的处理每一个文件。false就是正常的处理每个文件（该忽略的就忽略），适用于入口文件的任何引入文都有模块导出的情况。**

### 二、为不导出任何包的特殊文件设置sideEffect

之前提到过，**TreeShaking正常的处理文件的时候会对没有导出任何模块的文件进行忽略。**

**比如css文件\@babel/polyfill就是。他们并没有导出任何模块，因此会被treeShaking所忽略。但是我们不希望这样，这时候，我们就希望treeShaking不对某些文件进行处理**。

这时候就需要对sideEffect进行详细配置了：

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/de05f658b72cfd5d57a78e9f885fc7a0.png)

然后我们在重新打包。

这时候我们发现，TreeShaking似乎并没有产生效果，minus模块照样被引入了进来。

这是因为我们在mode:development，**也就是开发环境下。TreeShaking尽管已经得到了配置，也依旧不会生效，**这是为了方便我们在开发环境下调试文件，容易找到各种错误。

事实上，尽管TreeShaking在开发环境下没有为我们去掉不该用的模块，但是他已经提示了我们有哪些模块会在生产环境下得到使用，那些没有得到使用。

进入出口文件main.js,跳转到最后一行。

![](/images/posts/2019-09-04-webpack-×webpack_TreeShaking/2d53428143d941eef07ad49c01cb064a.png)

这就可以看出，TreeShaking的分析结果，他认为一共有两个模块，但是只是用了add。如果在生产环境下，就会只引入add。

### 三、生产环境下的注意事项：

首先mode需要改为:production

另外，我们的sourceMap也需要更改，不能存在eval，最好使用cheap-module-source-map.

而不是我们在开发环境下推荐的cheap-module-eval-source-map。

另外,optimization的配置也可以被注释掉。

但是package.json下的sideEffect还是需要配置的。
