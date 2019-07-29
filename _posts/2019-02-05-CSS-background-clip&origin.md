---
layout: post
title: backgroud-clip & background-origin的微妙关系实验
tag: Sass/Less/CSS
---

### 前言
background-clip和background-origin之间的关系总是这么的微妙。今天就来做一个实验来找到他们的关系。

### 实验思路
同一个dom结构下对应不同的background-clip和background-origin。值取他们的排列组合。

### 实验效果
![](/images/posts/background-clip&origin/1.png)
![](/images/posts/background-clip&origin/2.png)
![](/images/posts/background-clip&origin/3.png)
![](/images/posts/background-clip&origin/4.png)

### 实验源码
```html
<h1>background-color: pink</h1>
	<div class = "bg-image">normal</div>
	<div class = "bg-image border-box">border-box</div>
	<div class = "bg-image border-box" style = "background-clip: padding-box;">
		background-origin:border-box;<br>
		background-clip: padding-box;
	</div>
	<div class = "bg-image padding-box">padding-box</div>
	<div class = "bg-image content-box">content-box</div>
	<div class = "bg-image mixin-box">
		background-origin:content-box;<br>
		background-clip: padding-box;
	</div>
	<div class = "bg-image mixin-box no-repeat">
		background-origin:content-box;<br>
		background-clip: padding-box;<br>
		background-repeat: no-repeat;
	</div>
```
```css
.bg-image {
			height: 300px;
			width: 550px;
			margin-bottom: 40px;
			padding: 30px;
			font-weight: bold;
			color:blue;
			border: 15px solid rgba(0,.2,.3,.4);
			border-radius: 5px;
			box-shadow: 0px 0px 5px #888888;	
			background-color:pink;
			background-image:url('./img/3.png');
			background-size: 100% 100%;
		}
		.border-box {
			background-origin:border-box;
			background-clip: border-box;
		}
		.padding-box {
			background-origin:padding-box;
			background-clip: padding-box;
		}
		.content-box {
			background-origin:content-box;
			background-clip: content-box;
		}
		.mixin-box {
			background-origin:content-box;
			background-clip: padding-box;
		}
		.no-repeat {
			background-repeat: no-repeat;
		}
```
### 总结
1、背景图片两个关系比较模糊的属性：background-origin:border-box | padding-box | content-box;<br>
background-clip: border-box | padding-box | content-box;
2、background-clip:如果是repeat,则规定区域内必定存在图片内容。
		background-origin:只负责图片开始展示的位置和结束的位置。
		假设background-origin为content-box，那么content-box的左上角开始展示图片，图片的填充区域一直会被拉到content-box的右下角。
		background-clip顾名思义，背景的裁剪,但是他也有填充的特性。如果此时background-clip为border-box或者padding-box，那么因为图片开始展示的区域是content-box小于clip指定的范围，那么他就会对相比content-box多出的部分进行图片的填充（前提是repeat开启，否则就直接暴露出背景色了）
