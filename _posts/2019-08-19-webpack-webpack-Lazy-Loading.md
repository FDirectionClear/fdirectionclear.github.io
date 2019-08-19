---
layout: post
title: webpack lazyLoading 懒加载
date: 2019-08-19 
tag: webpack
---

webpack之Lazy Loading懒加载
===========================

### 懒加载是什么？

懒加载其实就是异步import语句。

webpack可以非常智能的识别出异步import语句，然后将需要分割的代码进行分割打包至生产环境。

然后在需要的时候引入生产环境下的已经打包好的文件。

具体的实现方式就是在需要的时候向head内插入script引用。比如触发事件，事件中存在import。（htmlPlugin生成的文件中并没有这个script引用）。

如果import的内容是同步的，那么webpack在打包的时候htmlPlugin就会直接在生成的文件中引用这个Import的打包好的文件。

看下面的例子：

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/b36b5aa3f46c5f739d41f621e267e31c.png)

我们可以发现在index.html中是这样的：

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/b410803a446d41dd20e6b8b36826ee3b.png)

换成异步的import再次尝试：

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/87be4b95652054f6cd99f2d28c92f7b3.png)

回看index.html，发现只存在一个main.js，而不存在lodash.js。

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/8a36407f561715c5cba32a37bdda34eb.png)

打开index.html的控制台

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/7392a0c5e9ae53fdfb83ab1364be147a.png)

发现还是没有在任何地方引入lodash.js。

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/9c0496e03408b98ca14dd40e2614454d.png)

network中也没有对lodash.js的请求记录。

然后点击body。

![](/images/posts/2019-08-19-webpack-webpack-Lazy-Loading/ea989bd2d52c07cfe70d4ab605901482.png)

此时，点击事件被触发，异步import导入lodash.js。然后lodash.js以script标签的形式插入到了head中。

此时network中也自然出现了对lodash.js的请求。

（一个失误，由于忘记修改分割文件的命名，导致vendors.js命名了分割出的lodash文件）。

这个就是懒加载的原理。
