---
layout: post
title: file-loader Vs url-loader及开发中遇见的坑
date: 2019-03-09 
tag: webpack
---

### 一、file-loader与url-loader

当我们打包一些非js模块的时候，比如jpg，png我们就需要去调查一下打包此类文件需要什么loader。

想要知道用什么loader，最佳的方式就是进入官方文档进行查阅，当然也有一些loader是在github上的，最知名的如:
clean-html-plugin。

![](/images/posts/2019-04-12-webpack-file&url-loader/16c50836fafa9e3c57cf927616dc03a7.png)

每种Loader的配置都有很多说道，想要完全掌握是不可能的，网页左边就是官方推荐的一群loader，点开就可以查看详细配置说明。

### 二、file-loader Vs url-loader 

![](/images/posts/2019-04-12-webpack-file&url-loader/e0e973f2c64416b1eee247d6d3dec3ea.png)

![](/images/posts/2019-04-12-webpack-file&url-loader/f4e93d70de351248609c217a5d6d22fa.png)

![](/images/posts/2019-04-12-webpack-file&url-loader/fd68f273cfd0360525d8483a8ee89736.png)

file-loader会打包图片，打包后返回的是图片的名称（打包后的图片就会‘复制’一份到生产环境下，只不过文件可能是经过webpack处理过的，而且名字也是通过webpack配置文件中配置的，事实上这个经过打包过后，出现在生产环境下的图片和原来的图片就是不同的图片）。

这样打包在生产环境下的图片就成了我们将在项目中引用到而定图片，这也就意味着，如果是通过src
+
图片的名称的方式引用图片，会产生一次http请求。如果图片较多，请求的次数也一定会变多，因此会造成性能下降的问题。

因此我们可以尝试用url-loader来代替file-loader(往往都是使用url-loader)。
因为url-loader不仅仅可以实现返回图片名称（和file-loader效果一模一样），而且还能返回base64代码。如果能使用base64代码替代图片索引式的src就能减少一次http请求。

不过依旧存在缺点，如果图片较大，生成的base64就会非常的长，导致js文件过大，加载缓慢。效果还不如直接用src索引进行http请求。

厉害的是我们可以通过限制图片大小，让url-loader智能判断到底应该是返回图片名称还是base64代码。

返回base64的样子（图片大小714kb）

![](/images/posts/2019-04-12-webpack-file&url-loader/f74891560f34d46373bdc6fe19431860.png)

![](/images/posts/2019-04-12-webpack-file&url-loader/d8fce7238cb103daf96df4c45b9560d0.png)

返回图片名称的样子

![](/images/posts/2019-04-12-webpack-file&url-loader/2fc5e90fb2fda34e8527d154d7863884.png)

### 三、补充说明

在.vue文件中，或者说在任何webpack的开发环境下，我们引入图片都应该按照以下方式：

![](/images/posts/2019-04-12-webpack-file&url-loader/6cd6f04f644e1482c66a4981164cbfa7.png)

而不应该是：

![](/images/posts/2019-04-12-webpack-file&url-loader/2de62754327df86851a010751176c9ef.png)

最常见的一个错误就是在开发环境下直接引入相对路径的静态资源。这样在绝大多数情况下都会失败，**因为webpack打包后的文件是在开发环境下，生产环境下对静态资源的引用相对路径很可能和开发环境不同！**

而通过import的方式，file-loader可以为我们返回字符串类型的相对路径地址。这个相对路径是针对开发环境而言的。

当然，如果非要通过自己手写字符串的方式，那我们就得确定好该静态资源相对于生产环境下的地址。
