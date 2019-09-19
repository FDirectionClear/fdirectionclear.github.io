---
layout: post
title: 懒加载与预加载
date: 2019-09-19
tag: 性能优化
---

懒加载
------

一般都是对于图片而言，第图片进行懒加载处理。也就是说当图片滚动到视口中时，将img的src属性填上，这时候浏览器就会在图片出现在视口中时才会进行http请求。

一般情况下，img的图片链接会储存在data-origin的DOM特性中。需要的时候把data-origin值拿出来赋值给src特性。然后在移除data-origin属性。

代码实现：

![](/images/posts/2019-09-19-XNYH-lazy-loading-and-preloading/8d7f8242003051a92115713540ba4683.png)

预加载
------

预加载的应用场景与懒加载正好相反，预加载是在网络空闲的时候提前加载好我们需要的图片。等到需要的时候就会直接从内存中取出，这样可以优化用户体验。

原生的预加载基本上有两种可选的方式，第一种方式是使用Ajax的方式请求需要预加载的图片，另一种是通过创建image对象，然后设置src的方式。

两种方式各有优缺点，Ajax请求的最大弊端就是跨域无法无法避免，优点就是可以通过各种ajax提供的接口和回调更活的控制请求各个阶段的行为。而使用简单创建Image对象的方式，虽然解决了问题，但是却很难对请求的各个阶段的行为进行把控。而且Image对象的创建也只局限于图片资源，如果需要加载的对象不是DOM中提供的对象，就无法通过这种方式实现预加载。

以下是预处理的两种方式的简单使用：

![](/images/posts/2019-09-19-XNYH-lazy-loading-and-preloading/01760cc1b26bf39e031d109763ae7a84.png)

![](/images/posts/2019-09-19-XNYH-lazy-loading-and-preloading/345a6401f529be49909be6a9c01c1d02.png)

效果：

![](/images/posts/2019-09-19-XNYH-lazy-loading-and-preloading/7616899b87b89d4fcf9ad4b6e7adb82b.png)

![](/images/posts/2019-09-19-XNYH-lazy-loading-and-preloading/c2de377dcb05bb114441ae3a4cec030a.png)
