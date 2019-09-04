---
layout: post
title: CSS模块化
date: 2019-09-04
tag: webpack
---

CSS模块化
=========

### 一、为什么我们需要模块化CSS

一般情况下css都是全局性的。如果仅仅在index.js一样引用.scss文件，会使所有文档内容都受到这个scss文件中的样式影响。举个例子：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/6f13504c6898e5b7bb36f98ddbfeb24e.png)

假如现在在index.js的同级目录下新建一个avatar.js

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/5af8769b3604f46070cca39e502cf7cf.png)

编写avatar.js

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/537e170bfae5e142e3b27ec77792fa94.png)

在index.js中引用createAvatar方法:

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/4c089af69e491279a9b1e3990f58d9b4.png)

**很明显，由scss引入的css样式就同时会作用于这两个图片。**

但是我并不希望这样，我希望在不为两个图片设置不同类名的情况下，也不修改scss的选择器。

**这时候就应该打开css-loader的模块化功能。**

*TIP:*

*关于css模块化(css-loader)的一些知识，可以查看阮一峰老师的博客,老师写的很好。*<http://www.ruanyifeng.com/blog/2016/06/css_modules.html>

### 二、开启css-loader的module选项

为css-loader添加选项：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/4b190a5bccac8c7feb7113bffd25b671.png)

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/7fa26e7f554fa2afc2a5cba3024b9c7c.png)

修改index.js代码，为只需要应用.avatar的元素进行如下操作：

**注意：因为把css作为一个模块，因此需要一个变量来接受css.**

**而不能直接import ‘../.scss’;**

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/7d7f9f4ce0bda0c34b1ec8bfd7a08ab0.png)

*TIP：他的原理就是把同一个模块下的所有类选择器都变成了一个独一无二的hash，之后将每个选择器的值作为这个对象中的key，变化后的hash（String）作为值，最后导出这个对象。*

*为了描述的更清晰，我们来看对于以下scss文件：*

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/2c845a60ed9ec58c859c5e1305abdca3.png)

*在index.js下引入，并尝试打印打出的内容：*

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/ceae378d1ddc35121b9abd28ea4dbc5c.png)

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/54c847d36c95a3d9e6abc57b06ad0587.png)

查看index.html下的head标签中的内容：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/6613c2796fe1870ca6b66c73e9c17233.png)

*另外还需要注意的是，元素选择器并不会被hash化，否则元素选择器就会变得毫无意义不是么？*

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/bcec688e131321f6389d47dc867bd601.png)

再次查看打印结果，发现毫无变化：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/de647bc685e00d7cc444bd12032c0f9b.png)

\---------------------------------------------------------------------------------------------------------------------------------

最后就可以发现，在index.html的style标签中的代码变成了这个样子。

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/181a02fec92ae205a021d43c0af4fb89.png)

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/9b56b91553b0843785c6838b891358c2.png)

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/3e32c6d5fbf16e8bb9f334ba6d644b77.png)

由此可见，为了实现css局部作用，只不过是为每一个css模块下的选择起得值转换成为一个独一无二的hash值。即使是同一套样式代码，在不同模块下的hash值却是不同的。因此，这也就实现了同一个模块下的选择器总是独一无二的，作用域的范围也就被限定了下来。

### 三、实战中遇到的坑点

来看下面这个坑点：

如果scss的文件是这样的：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/1e950c2126d790eab97ee4f5854d4aed.png)

其他不变，最后编译在index.html中的内容却是这样的。

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/fc278fe69ee1b9cc6908cb17621922dc.png)

呈现的效果：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/fe4739a171b6f9f9b7fe786e87ae0d06.png)

还是根据之前的补充说明，，只是把需要转换为局部变量的css属性换成了hash。其他没有要求的（也就是没有style.xxx）的属性依旧不会变化，还是会被style-loader原封不动的添加在了style标签内。元素选择器不会被hash化！因此第二张图片还是受到了img选择器的影响。

### .vue中遇到的坑点

在.vue文件中启用css
modules可能会造成不必要的混乱。我们都知道.vue中对局部作用域样式（scoped）的处理是通过为当前.vue文件中的每一个dom节点加一个与其他.vue文件中不同的hash值特性。然后为.vue文件中的每一个选择器后都加和这个hash值对应的属性选择器，一次来达到样式模块化的作用。但是值得注意的是，.vue对样式模块化的处理并不是通过修改选择器的值，他并没有修改选择器的值，如果选择器的值发生了变化，那么样式就无法和template模板中dom的class相匹配了。最后会导致.vue中描述的样式根本无法成功渲染在组件上，这样就造成了不必要的混乱。

**翻车现场：**

我的某个SPA项目中的style标签中的内容

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/0f21ce3740b89285981fbfdb38af3126.png)

DOM结构：

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/f00c6b1f31f0a2325cb1215c4b0d82bc.png)

所以就造成了结构的大混乱。

![](/images/posts/2019-09-04-×webpack-webpackMoKuaiHua/c7d1eaeae827188eeb26cfb2f9982558.png)
