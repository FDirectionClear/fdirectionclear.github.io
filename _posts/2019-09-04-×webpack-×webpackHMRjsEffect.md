---
layout: post
title: Webpack HMR自动修改js效果
date: 2019-09-04
tag: webpack
---

Webpack HMR自动修改js效果
=========================

上次说道HMR修改css文件可实时的反应当前页面渲染而不作出任何的浏览器刷新。

但是对于Js文件就无法做到这一点。那么这次就专门对js的HMR作出探究。

### 一、js默认HMR无效的例子

同样是打开HMR，如果修改了Js代码是不会自动改变的。

实验过程如下：

**1、部署基本逻辑**

![](/images/posts/2019-09-04-×webpack-×webpackHMRjsEffect/5eb8d179c540def2c8ab14ef1bbc2c5e.png)

![](/images/posts/2019-09-04-×webpack-×webpackHMRjsEffect/fcd8429185db2c9a21b6398dda98fea1.png)

TIP:

为了清晰的描述不刷新浏览器的过程，特引入了count()方法，他的作用是点击7000上面的那个数字会自动递增。现在点击了13次，显示蜀滋味13。如果此处刷新浏览器13将重置为0。

![](/images/posts/2019-09-04-×webpack-×webpackHMRjsEffect/119cacf2644a3de93cd6e5fd0caeafcb.png)

**2、修改number.js;**

![](/images/posts/2019-09-04-×webpack-×webpackHMRjsEffect/11a6fea2d4c10fe6457fa13e671175cb.png)

**3、不刷新浏览器。再回来看一眼浏览器变化**

![](/images/posts/2019-09-04-×webpack-×webpackHMRjsEffect/370d02942d5c9b9fcca99173a3fa9704.png)

如果此时js有HMR的能力，那么会在不刷新浏览器的情况下，13自动变化为1000。可是现在并没有变化，说明js目前还不支持HMR。（注意13没变化，说明我们没有刷新浏览器！）

### 二、使用HMR的相关API让js也热更新起来

webpack的HMR并非不支持js文件。webpack为我们提供了很多关于HMR的接口，让我们自行实现热更新的逻辑。为了解决js热更新的这个问题，我们需要对文件的改变做出一个监听，同时给予监听的逻辑。

在index.js下：

![](/images/posts/2019-09-04-×webpack-×webpackHMRjsEffect/b46880cacb099e23e46c7c89a29c6aaf.png)

在这里需要注意的是，修改了index.js,根据浏览器控制台的提示信息，需要reload整个文件。因此重新npm
run start;
之后就可以实现多次点击上面的数字，修改number.js还能保证在不刷新浏览器的情况下更新下面的数字，而不影响上面点击后的数字。

但是在一些高级的框架中，比如Vue，vue-loader已经帮我们写好了module.hot.accept的部分了。

其实在我们更改sass和css文件的时候也会触发module.hot.accept方法。只不过是webpack内部已经帮我们实现了相关逻辑，这也就是为什么HMR默认对css等样式文件起效的原因。所以一般情况下不需要重新编写。
