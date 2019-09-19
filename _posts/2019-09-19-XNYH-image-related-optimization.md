---
layout: post
title: 图片相关的优化
date: 2019-09-19
tag: 性能优化
---

图片上的选择
------------

png图片使我们做网站的时候最常用的图片格式之一。png的类型又主要分为三种：

1.  png8 ——256色 + 支持透明 每一个颜色值的索引只需要2\^8个比特

2.  Png24 —— 2\^24色 + 不支持透明
    每一个颜色值的索引需要2\^8个比特，每一个颜色的长度是png8的3倍。并且不支持透明。

3.  Png32 —— 2\^24色 + 支持透明
    在png24的基础上为每个颜色值的索引增加了8比特去让它支持透明。

虽然颜色的索引越多图片就会更清晰，颜色更鲜明，但是这也会增加图片的大小，设想这样一个需求，一个不是很需要清晰度的纯色图片或者接近纯色的图片：比如大海，蓝天。如果png8就已经能满足需求那为什么一定要更大的png24或者png32呢？

不同格式图片常用的业务场景
--------------------------

1.  jpg有损压缩，压缩率高，但是不支持透明。**所以更适合大部分不需要透明图片的业务场景。**

2.  png有上面说的很多规格可以选择，支持透明，并且浏览器的兼容性很好。**使用在需要在透明图片的场景。**

3.  webp是google在2010年提出的图片格式，压缩程度更好，支持透明，但是在IOS
    webview有兼容性问题。

4.  svg矢量图，代码内嵌，相对较小，因为是作为代码嵌入在html中的。所以性能更佳，但是因为是代码写出来的，所以绘制能力有限。有iconfont这样成熟的svg素材库。**如果小图标能用尽量用svg。**

图片压缩的几种方法
------------------

#### 1、雪碧图

>   雪碧图是老生常谈的一种压缩图片的方式，原理就是通过吧所有小图标做成一个大图，然后在需要某张图片的时候通过定位去从这张大图中割取。旨在减少http的请求次数。

>   但是同时也带来一些问题：

1.  合成的一个大图可能太大，http虽然只需要请求一次图片，但是图片大还不容易加载。

2.  如果图片没有加载完成，或者加载失败，那么整个网站需要用到雪碧图的地方就全部沦陷。

*TIP:
自己动手使用雪碧图可以通过ps将我们需要的小图标不重合的拼在一个大图里面，然后使用在线雪碧图生网站www.spritecow.com去导入我们制作好的雪碧图，这个网站会告诉我们雪碧图各个位置的css属性，包括定位的位置。*

#### Image inline

Image
inline是非常常用的一种方式，可以完全不必单独向服务器发送图片请求。因为是以base64的方式插入图片的src特性中，所以只是额外增加了一些html的文件的大小，不过html的请求是在主干网络上的（或者说下载html时没有其他资源一起下载），所以总的来数，相比多次进行http请求，嵌入在html里，和html文件一起在主干网络上加载还是很划算的。

但是缺点也很明显：base64的代码量很长，如果是小一点的图片还好，高清大图巨长，可能会在储存和编辑上带来麻烦。

#### 借助在线网站

直接借助在线网站进行图片的压缩，这种方式明显不实用，但是确实是一种图片压缩的方式。

比如：tinypng.com