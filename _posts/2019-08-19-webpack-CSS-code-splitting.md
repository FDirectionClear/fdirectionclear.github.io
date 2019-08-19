---
layout: post
title: Webpack CSS代码分割
date: 2019-06-15 
tag: webpack
---

### output.chunkFilename和filename和vendors.name以及魔法注释webpackChunkName的区别

先只使用魔法注释，其他的均不设置，来查看效果：

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/2b149b85c4686d2463161624c874e3ad.png)

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/2b6866586114423c4be0283d8c5bfe9f.png)

我们可以看到除了main.js这个入口文件外，还打包出了两个js文件，第一个是vendors\~lodash，第二个是vendors\~main（并没有在Index.js中引入这个文件，但是还是打包出了，怀疑是babel的core-js\@3？）。

vendors说明这两个js文件他们都符合vendors组的条件，但是由于vendors组没有设置统一的name，所以符合vendors组的内的文件有一个分割一个，而不是统一分割在一个文件中。

另外可以看到魔法注释事实上就是决定了chunk
Name的vendors\~后的名字（因为在默认情况下都符合vendors组的条件），而打包出来的js文件的名字就是以chunk
Name命名的。

这时我们为vendors.name设置一个值“fang”。

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/13e0e59256589f01c01b373e6857e550.png)

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/2e4517ddb3356ff4594a6e32ea8d493d.png)

我们可以发现，由于设置了vendors组内的所有文件的统一名字”name”，原本的vendors\~lodash和vendors\~main都被统一合并打包到了fang这个Chunk中，然后这个js的文件名依旧以Chunk
Name命名。值得注意的是魔法注释已经被忽视了。

接下来我们在设置一下output.chunkFilename:

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/7a3b6a419a0dcc45e55d0ec6ddfb664b.png)

可以看到，output中同时存在filename和chunkfilename，那打包出的文件应该遵从哪个呢？

尝试打包：

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/c1c435ea8abbce1c744153bbdc32c9e2.png)

由此可见，filename是用来设置各个入口文件的打包文件名的。而chunkFilename则是用来设置从各个入口文件下分割出来的文件的文件名的，每一个分割文件都是一个Chunk，其中占位符[name]就是对应的chunkName。

总的来说，被分割出来的Chunks都是以自己ChunkName命名的，这个ChunkName是由魔法注释或者是optimization.splitChunks中设置的。如果没有设置output.chunkFilename，那么现在的这个名字就是最终的名字，如果有设置output.chunkFilename，那么这个名字就会被再次包装（一般都用[name].xxx.js）成为最终的文件名。

当所有的入口文件中分割出好多个Chunk文件时，output中只有一个固定filename,则会发生报错。

此时解除vendors.name，让他分割出两个Chunk。

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/e85105dd031f0690b2515b9746299365.png)

然后设置output.chunkfilename:chunk.js后进行打包，webpack就会显示报错：

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/db0ce921a9f9e0f504952909297845bd.png)

### webpack分割CSS文件

想要分割css文件，就需要借助插件mini-css-extract-plugin

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/8a345f909886938aea5d0bf7100e3f5f.png)

需要注意的是，treeshaking会把index.js中引入的css文件直接干掉，所以我们应该现在package.json中配置好sideEffect。（先跟将TreeShaking的配置移动到common环境下，之后我尝试将sideEffect去掉，或者设置为false，mini-css-extract-plugin都能正常的作出css分割。不知道为什么？）

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/8f8fe22d589db93b3191a2c6bd857587.png)

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/c39896577eb532ba9be7b777073bd232.png)

然后就会发现css代码被分割出去,index.html中也自动link了这个main.css文件。

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/dda8e5e3e31538ed87d08a71cc7f6185.png)

在mini-css-extract-plugin中也有很多配置：

比如filename和chunkFilename就是决定打包生成的css文件的文件名称的。那filename和chunkFilename的区别是什么？

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/c73970a6133e6939d6216070e624beb9.png)

**补充：**

**然后将style-loader替换为MiniCssExtractPlugin.loader，如果要分割sass/less/stylus也是一样，只要把style-loader替换即可。**

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/ccc60b46a9849810ffe8a3b7484bfe20.png)

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/6103a7d8d7e0cbefae742ad567a71170.png)

所以可以看到，mini-css-extract-plugin可以将同一个入口文件中所有同步引入的css打包至同一个css文件中。

但是此时还没有满足我们的需求，打包出来的css文件没有被自动压缩，如果我们需要压缩css，我们就需要使用另一个官方提供的插件。

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/da678b0770331f5940079c77db0f20b4.png)

具体配置方法极其不容易记住，就参阅文档吧。

<https://webpack.js.org/plugins/mini-css-extract-plugin/>

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/9c774eb46976d0f51ad15e63c2ab7751.png)

压缩之后的css代码张这个样子：

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/4825320d0fb0bb58315e728fcb630f05.png)

![](/images/posts/2019-08-19-webpack-CSS-code-splitting/de8cc36666db9aa81a64786602de0c77.png)

关于css的代码分割，底层其实也是经过splitChunks的一些配置的，这些配置可以决定很多打包css的方式，比如多个入口文件的css都打包到一个css文件中等。

详见官网：

<https://webpack.js.org/plugins/mini-css-extract-plugin/>

下图是多个css打包进一个css文件中的基础配置，可以看到其实也是通过splitChunks实现的：

![](me/images/posts/2019-08-19-webpack-CSS-code-splittingdia/5a76a8b21df0f7df159b80b8e9864c81.png)
