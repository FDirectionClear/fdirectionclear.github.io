---
layout: post
title: Webpack 性能优化
date: 2019-09-04
tag: webpack
---

Webpack 性能优化
================

跟上技术的迭代
--------------

无论是webpack的升级还是node、npm、yarn、之类的升级，内部肯定有考虑到性能优化的问题，所以跟上技术迭代无疑可以增加webpack的打包速度。

在尽可能少的模块上应用loader
----------------------------

有的模块能不用就不用，或者有一些根本就没必要用。比如babel-loader处理js文件，就可以excludes:/node_modules/。

TIPS:并非所有Loader都是用excludes。

Plugin尽可能的精简并确保可靠
----------------------------

比如我们需要在生产环境下使用optimizeCSSAssetsPlugin()来压缩CSS，但是在开发环境下这个就没有必要了。

resolve参数合理的配置
---------------------

### 1、resolve.extensionsd

我们都知道，.js文件在引入的时候可以不加后缀，但是有些时候，我们希望webpack能对其他不加后缀的文件也进行识别，那么我们就可以配置resolve选项。

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/52d0b202785601b74f3f051bf186314c.png)

Resolve.extensions是一个数组，数组的每一项就是可以省略的后缀名。比如按照上面这样配置我们就可以省略.css、.jpg、.js、.jsx的文件。这样看上去给我们的开发带来了很多便利，但是事实上却存在着打包性能的浪费。

设想，我们引入了一个图片文件，文件名为picture，并且这个文件并不存在于指定的文件夹下。那么webpack在打包的时候会先按resulve.extensions中设置的后缀名的顺序依次进行寻找，直到找到为止，可是直到找完.jsx文件才发现根本没有。这个寻找的过程重复了整整4遍，最后却捞得一场空。

所以我们不能滥用resolve，最好还是写上后缀名，对resolve.extensions的配置最好也只限于js和jsx文件。

### 2、resolve.mainFiles

当然resolve可以配置的内容有很多，如果我们只希望引用一个文件目录，就可以自动引入这个文件目录下的某个文件，我们可以配置resolve.mainFiles。

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/b1e14fe35d58846d9c4382b8aefa5240.png)

这样我们就可以这样引入文件了：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/d7d342fe6af15d9fb90caf76ca66666e.png)

事实上，这个mainFiles的默认值是[‘index’]，也就是说我们在不显示配置mainfile的情况下，引入一个文件目录，是会自动寻找**名为index的文件名**的文件的。

因此，这样引入文件是会带来同样的负面影响。

### resolve.alias

alias的意思是“别名”，alias是一个非常常用的配置项，当某个模块的文件路径非常复杂的时候，可以通过为这个路径配置别名从而获得这个路径的“语法糖”。

比如当我们想要获取index.jsx文件的时候，就可以这样配置（可见jsx的路径的复杂度还是很高的。）：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/2117e24c77e6a6e2784b9b4528725594.png)

引用的时候就可以这样：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/18d3d2eba4be29502611f2aefb5a9e96.png)

### 使用dll优化打包速度

在每次打包的时候，webpack都会所有需要打包的文件进行分析，其中包括一些静态文件。

这些静态文件通常上是不会改变的。因此webpack的每次打包都重新分析这些静态文件是一种性能上的浪费，如何才能避免这样的浪费呢？

我们可以创建一个独立的webpack.dll.js，设置mode为production即可，然后在这个文件中，对我们需要的静态文件单独进行打包。

在配置中，可以看到我们将react、react-dom打包到了react.dll.js，lodash。然后将其输出在dll目录下。然后又通过library将react、react-dom以及lodash的内容全部暴露在了全局变量（react、lodash）下。

为什么需要这么做？

这是因为我们希望能通过一次手动打包静态文件来解决以后的所有打包这些静态文件问题，大体上就是，我先额外打包好静态文件，然后每次打包的时候，都会发现依赖的静态文件已经打包好并放置在了dll目录下，所以我们只需要对这些已经成型的静态文件的打包成品进行一些处理就可以引用的到他们。

具体的操作步骤为：

1.  配置webpack.dll.js，打包静态文件，同时利用library将静态文件的内容绑定到全局变量上。

2.  在打包静态文件后，通过library将个静态模块的内容暴露在全局变量上。

3.  在webpack.dll.js中使用webpack.DllPlugin()来分析library暴露的全局变量，从而导出静态文件的映射文件[name].manifest.json

4.  在webpack.common.js中通过add-asset-html-webpack-plugin将webpack.dll.js打包好的js文件在index.html中自动引入。

5.  在webpack.common.js中配置，让webpack.DllReferencePlugin结合各个[name].manifest.json文件分析打包中依赖的各种文件。当发现需要打包的import的文件在dll目录下有现成的打包文件时，就直接引用，而不用重新打包。

示例图片如下：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/35e8ce46ae06958ec8cc2c3ce766ff08.png)

打包后生成文件目录：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/d8ac1a0bb043652afaeba24dd511a171.png)

打开lodash.dll.js和vendors.dll.js就可以发先library都做了什么：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/addb6d9a442caee1b7f99cf219154552.png)

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/5f6e96c537e806c9a567faa4602fa4fd.png)

配置common.js

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/8d106ec69566a9e81bcf7cf748bdd490.png)

重新进行打包即可发现打包速度加快。

Dll改进前：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/1c53e5ce10aac89b21660d209a2bebe0.png)

Dll改进后：

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/0234be69f71ccaff084ce29e5ae204ca.png)

### dll的使用增强方法

当我们做一些大型项目的时候，可能会有很多的静态文件，如果我们一个一个的通过addAssetHtmlWebpackPlugin和webpack.DllReferencePlugin去添加关系会非常的麻烦，那我们该如何让dll变得更智能呢？

我们可以用Node去读取dll目录下的所有文件，然后将文件名存入一个数组之中，然后通过遍历数组，对不同的文件名用不同的plugin去处理即可。这样我们在增加静态文件的时候，只需要增加dll的入口文件即可。

![](/images/posts/2019-09-04-×webpack-webpack _webpack_XingNengYouHua(1)/731c7cba861a386787c786f0926ad1bd.png)

可以看到，我们直接将plugins选项分离出来，然后通过fs模块读取文件，在遍历文件数组，通过正则表达式去判断文件的类型，在根据类型通过不同插件去处理它，最后全部push进进入plugins中就OK了。
