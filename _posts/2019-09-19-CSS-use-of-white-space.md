---
layout: post
title: White-space的使用
date: 2019-09-19
tag: html&&css
---

今天写项目的时候遇见了一个问题，感觉这个问题我以前遇到过而且很简单的就解决了，但是如今又碰到了这个问题却忘记了。

当我们想要一个父元素的宽度固定时，里面的元素就会尽可能的占满一行，如果内容太多一行装不下就会自动换行，但是有些时候我们不希望换行，这时候我们就需要white-space这个属性了；

我想要实现一个点击左右箭头可以滑动的时间轴。

![](/images/posts/2019-09-19-CSS-use-of-white-space/4297bc79d0f7881147570b368b92f5a2.png)

上面代码实现的效果是这样子的，注意white-space被注释了。呈现的效果如下:

![](/images/posts/2019-09-19-CSS-use-of-white-space/c395329942b1e0eb5d66c2f848c4ce08.png)

上图中，绿色边框是父元素，固定了宽高，里面的红色边框是子元素，同样也固定了宽高。当子元素（红色边框）的个数较少的时候，也就是绿色父元素的宽度能容纳所有红色盒子的宽度总和的时候不会出现什么问题，也不会发生换行。但是如果子元素过多就会出现问题。

![](/images/posts/2019-09-19-CSS-use-of-white-space/6d8bf9b2b2bbfc4598a962b55500294d.png)

但是如果解除white-space的注释；就会得到以下效果：

![](/images/posts/2019-09-19-CSS-use-of-white-space/f622a400070848f3ef3db854881270bf.png)

绿色框有设置为overflow:auto；

可见红色的子元素并没有发生换行。

接下来回顾这个white-space属性；

![](/images/posts/2019-09-19-CSS-use-of-white-space/f46dd93c4db395eb59f4bda45c7a46e6.png)

White-space:normal \|\| pre \|\| nowrap \|\| pre-wrap \|\| pre-line

值得注意的是，这个属性只对是inline和inline-block的元素有效；

因此如果内部的元素是块状元素的话，要记得设置为inline-block；float自然不行。

![](/images/posts/2019-09-19-CSS-use-of-white-space/a8f89fcd3da1fc0f7521773b6280efcd.png)

Pre-wrap \|\| pre （经过我的实验没发现什么区别，有待研究）效果如下：

![](/images/posts/2019-09-19-CSS-use-of-white-space/cd9b423e158df74ea03929a34fc2440f.png)

Pre-line 效果如下

![](/images/posts/2019-09-19-CSS-use-of-white-space/4a18b52922785173a29ff57427ac51c9.png)

html代码皆为：

![](/images/posts/2019-09-19-CSS-use-of-white-space/6956669c7ddd5908813d6a3120c6ed1a.png)
