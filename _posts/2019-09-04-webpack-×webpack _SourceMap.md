---
layout: post
title: Webpack Source Map
date: 2019-09-04
tag: webpack
---

Webpack Source Map
==================

如果存在文件中的代码错误，控制台报错说打包后文件（这里是main.js）的哪一行出现了错误，而不是告诉我们真正的那个入口文件的哪一行出现了错误。这对我们的代码调试极其不便利。

Content.js:

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/0c60c16669e337b4f5e8769da68f9c8e.png)

控制台报错：

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/8c46f27bcb3398e7a7c5f9d37a995c13.png)

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/2aec2bf4e66955659639708ad76c0fa3.png)

如果希望映射出错误文件的具体位置。需要打开sourceMap功能。

**还有一点需要注意的是，在mode为开发环“development”时候，**

**sourceMap功能是默认打开的。**

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/e7734ec0f4a6d7447aa1a3cb11071790.png)

如果打开sourceMap功能：

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/85dab8fef49e9420143abd1224b0e893.png)

报错信息就会换成文件的映射：

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/a619e8ce1ccd13bbf4633d265946537b.png)

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/14999ecb20370876ae46f1778b4ab67b.png)

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/ce27ebff11a4035c2c3ed3bcb09532e5.png)

devtool也有很多关于soucemap的设置，比如在**生产环境下我们常用的是**

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/b5337fc0deb924422c3b0904a8f914d0.png)

**在开发环境下网上最推荐的是：**

![](/images/posts/2019-09-04-webpack-×webpack _SourceMap/a2122212d6abfdbb78de79bf10bcae89.png)

这种设置会提示的全面且性能较好。

这些不同的设置都会产生不同的反应。会带来性能上的差异。

详细可以参阅官方文档。
