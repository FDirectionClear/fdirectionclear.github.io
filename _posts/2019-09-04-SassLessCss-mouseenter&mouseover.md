---
layout: post
title: onmouseover和onmouseenter的实验
date: 2019-09-04
tag: Sass/Less/CSS
---

时间长了，有点忘记onmouseenter和onmouseover的一些技术细节了，只记得一些关键的区别，就是他们对于子元素是否触发，之前有一段时间还特意研究过，这里重新做实验，算是拾遗。

### 对应关系

onmouseenter和onmouseleave对应

onmouseover和onmouseout对应

### 实验准备

1.  Html + CSS + JS

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/9d2533da255ad538b8ddc102000f1a55.png)

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/f169551d798da49e1e19b5b7a31d0cca.png)

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/f79ceaeb5660eca45f2edcd1763830ff.png)

### Onmouseenter

首先是onmouseenter：

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/949d587e5dc8aefbb88869ba362fd6be.png)

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/8787d7609c226a3889decbe0687ec168.png)

控制台只打印了一次，**说明onmouseenter是一个只被会父元素，而不会被其中的子元素触发。**与onmouseenter对应的还有onmouseeleave，也是这个道理。

### Onmouseover

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/eb060b58bb0a60fddf019584d7b6d8b3.png)

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/39b537edb27a7620eace3bdf7e10ef99.png)

虽然本次路径看上去和前一次不同，但是事实上是一样的。

再来看看控制台打印的结果：

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/fa9b8e9d593d5bfdb338d00d8031a41d.png)

可以发现，onmouseover事件在这个简单的移动中，被触发了5次！并且先parent在child1后又穿插了一次parent，之后进入child2被触发，然后又出发了一次Parent。那下图为parent

触发点：

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/427d4a30f7d4ba2282d4b43584859d47.png)

**那为什么会这样呢？**

原因很简单，如果用最直白的话说，就是父元素和子元素所占的地皮是独立的，而且onmouseover是监听的整个父元素及及其下的子元素。换句话说，子元素虽然在父元素中，但是子元素所在的区域已经和父元素完全独立开来，进入子元素就相当于脱离了父元素的区域，一旦离开子元素，就又会进入父元素的区域，所以中间就会穿插触发一次parent的onmouseover。

### 当子元素从父元素的图形所占区域中偏移出的情况

当父元素和子元素相隔甚远的时候，鼠标悬浮在子元素上，还会正常触发事件么？

修改css代码，使用各种方式为child1设置偏移。

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/9403a9fc7cd649f42ce1d3be95d18446.png)

先来看看postition的方式产生的偏移：

1.  onmouseenter：

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/4a3d7cce283748c2b6f4a0f0be8db1be.png)

#### [.//images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/image12.png](.//images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/image12.png)

1.  Onmouseover

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/e48d121fbdf575c0624b8bcd983f6e63.png)

![](/images/posts/2019-09-04-SassLessCss-mouseenter&mouseover/07254ea75236068bbf79484eaa9f9c1d.png)

**可以发现，无论是enter还是over都触发了两次。只不过对于onmouseenter触发的元素为父元素，而对于onmouseover触发的元素是子元素！**

### 结论总结

（1）onmouseover 和 onmouseout是区分子元素和父级元素的。

当为父元素设置onmouseout时，即使鼠标没有脱离父元素区域，当鼠标进入一个子元素区域时，就会触发，触发元素为parent。

当为父元素这是onmouseover时，即使子元素在父元素区域内，鼠标进入子元素区域依旧会独立触发一个。出发元素为child。

而 onmouseenter和 onmouseleave是惰性的，只对父元素有效。

（2）当子元素以任何偏移的方式（relative, absolute, margin, padding,
translate...）的脱离了父元素的图形内部，然后鼠标进入子元素时onmouseover和onmouseenter依旧还是会触发。但是区别还是有的，onmouseenter的e.target总是parent，而onmouseover就可以精准的判断出e.target是parent还是child。其实道理和结论（1）是一样的。

（3）onmouseover也好还是onmouseenter，以及他们对应的离开事件都是根据元素的实际大小和实际位置而定的。
