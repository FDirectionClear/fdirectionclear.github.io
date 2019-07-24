---
layout: post
title: babel-loader配置
date: 2019-06-16 
tag: webpack
---

### babel初体验

进入babel的官网，然后在导航上点击使用，选择一个执行环境，这里是webpack，

无论是什么环境使用babel，下面都会有清晰的教程。

进入下面链接即可复习观看（babel中文文档）：

<https://babel.docschina.org/setup#installation>

需要注意的是，安装了babel-loader还没完，他并不会对我们的js代码进行翻译。这只能说是打通了webpack和babel之间的桥梁。还需要下载转码规则之类的东西，配置好文件才能进行“翻译”。

*补充说明：*

*另外，只下载babel-loader还是不够的，还需要下载\@babel/core、\@babel/preset-env、\@babel-polyfill。他们分别是babel的核心，转码规则，和对浏览器解析器和引擎做的一些补丁babel-polyfill，比如ES6
中的promise，他无论如何都无法通过ES5实现，这就必须对浏览器js引擎上做手脚。当然polyfill和preset-env也是可以按需选择的。*

具体还可以做的事情有很多，比如选择各种转换规则等，官网写的很清楚。在这里不多做笔记了。babel的官网非常友好，一看就会。

在完成babel的安装相关后，我们就可以开始对babel做一些配置了。配置可以直接写在babel-loader的options选项中，也可以独立出一个json文件.babelrc。我们更推荐后者。

因为node_modules中的文件已经经过了ES6向ES5的语法转换，就不在需要babel进一步耗费时间和空间去再次解析。因此我们可以通过exclude（相反的还有include，他们的应用场景不仅仅在babel的配置中，只要需要特殊针对某些文件集合，往往就可以使用exclude和include）。需要注意的一点是exclude的写法，他是一个字面量的正则表达式。

![](/images/posts/2019-06-16-webpack-babel-loader/80bef834846f5a1217652fc01df7cbc7.png)

下载配置好babel就可以将ES6代码转换成ES5的代码了。

### 使用polyfill

但是这样子还是不够，因为他只是进行了语法的转变，有一些对象和方法ES5本身就没有，无法通过替换语法的形式做到转化。因此还需要借助插件，将ES5缺失的一些对象和方法补充进去才行。这时候，babel-polyfill闪亮登场。

<https://babel.docschina.org/docs/en/babel-polyfill>

配置完polyfill等相关内容后，即可打包。**注意如果使用polyfill需要把polyfill
import在所有文件的开头。**

![](/images/posts/2019-06-16-webpack-babel-loader/11fc95e8560f416ee279c0e234453d7b.png)

**补充说明：**

**如果我们配置了优化选项useBuiltIns: ‘usage’
那么polyfill不用显式的引用在入口文件的开头。他会自动分析出文件中需不需要额外扩充补丁，如果需要就会自动引入polyfill。当然,polyfill还是需要下载的！**

Babel-loader的配置内容options对象，他的内容可以完全的写在一个.babelrc的文件当中，.babelrc文件应该存在在整个项目的根目录部分。这样做更好。使得webpack和babel的配置分离开来。

![](/images/posts/2019-06-16-webpack-babel-loader/bd81ef601c5f1258a8cdaaf9750d5d51.png)

在写.babelrc文件的时候需要注意：

*TIPS：该文件名为.babelrc即可，注意文件内不能写注释，和json文件格式相同。所有的属性和所有的值都应该是用""来包裹。*

.babelrc:

![](/images/posts/2019-06-16-webpack-babel-loader/141d3ad420ab10a705c44a7118ae4c40.png)

Webpack.config.js:

![](/images/posts/2019-06-16-webpack-babel-loader/72ffdfbadf18efc48b470e2d973ab91c.png)

babel有很多讲究的配置参数，详细的配置浏览于官网。转码规则的配置算是最重要的了。

**还需要注意上面对转码规则配置参数的方式，他是一个二维数组！**
