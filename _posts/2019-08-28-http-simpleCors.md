---
layout: post
title: CORS跨域请求的限制与解决办法
date: 2019-08-28
tag: Http
---

因为出现了要用devServer开发的项目要向某个php文件发送请求的需求，如果一直使用Jsonp实现跨域请求感觉十分的不方便。这次就尝试通过服务端的角度来解决跨域问题。用node实现两个简单的服务器，然后通过CORS的方式实现跨域请求。

### Access-Control-allow-origin

原理很简单，我们只需要在消息头中设置好Access-Control-allow-origin属性即可实现跨域请求。

以下是我的实验现场：

1.  创建server.js，用来充当我们开发时所在的服务器

![](/images/posts/2019-08-28-http-simpleCors/38657aca956a1b9a4801a9b4c6ad5969.png)

其中server.js返回的test.html如下：

![](/images/posts/2019-08-28-http-simpleCors/8f3621f685173dc0c6086e6e458276de.png)

1.  创建server2.js，作为不同域下的服务器

![](/images/posts/2019-08-28-http-simpleCors/4ed8267c1bce32cfed19b36eff43febf.png)

>   如果成功请求到server2.js，那么就会返回字符串123

1.  启动两个node服务，同时访问localhost:8888

![](/images/posts/2019-08-28-http-simpleCors/10bac5c0e23d96046fd5fa4392066ff1.png)

![](/images/posts/2019-08-28-http-simpleCors/46e60cfffeea744811a2b1b37a98b68d.png)

因为我们两个node服务监听了两个不同的端口，一个是8888,而另一个所在为8887。所以就造成了跨域拦截。

1.  为server2.js设置Access-Control-Allow-Origin

![](/images/posts/2019-08-28-http-simpleCors/61ef6a55679c9f4b735decf4cd8d3026.png)

1.  再次访问localhost:8888

![](/images/posts/2019-08-28-http-simpleCors/cde5ecb5eb3d4bd333ba558981d68f4c.png)

>   可以发现请求成功返回。跨域的限制解除了。

>   从返回来消息头中我们可以看到Access-Control-Allow-Origin

![](/images/posts/2019-08-28-http-simpleCors/2f77b492c8c68fdf14bf3daff493759a.png)

>   当然：Access-Control-Allow-Origin属性的值如果是’\*’就代表这所有的网站都可以跨域访问到我们当前服务器上。如果我们不设置为\*，就可以以一个url字符串来替代。这样就能够指定那些地址可以跨域过来。

![](/images/posts/2019-08-28-http-simpleCors/960150da414deb28c3595db2b072167d.png)

>   返回依旧成功：

![](/images/posts/2019-08-28-http-simpleCors/84b0817aa9076635524ff792e48ba70d.png)

**坑点注意：**

**在这样的情况下，如果我们用localhost:8888去访问127.0.0.1:8887还是会出现跨域的报错，这是为什么呢？**

**原来浏览器并不知道localhost会被映射成127.0.0.1。所以依旧会认为请求来自不允许的域。如果没注意到这一点还真容易翻车，纠结.....**

**翻车现场：**

![](/images/posts/2019-08-28-http-simpleCors/c8ba4978ed24da7ae8b37bd38b7509ed.png)

![](/images/posts/2019-08-28-http-simpleCors/aa462ce2897a03f160086b90cc675be1.png)

### 补充说明

其实服务器并是不会理睬那些来访者是谁的，也就是说他根本不会关心来访者是不是跨域来的。我们向服务器发送的请求，无论来自哪里，服务器都照样执行逻辑，依旧返回应该返回的信息，而跨域而产生的拒绝是浏览器自主产生的。因为同源政策的存在，浏览器会在每一次消息返回的时候都检查我们的请求是否存在跨域现象，如果浏览器发现存在跨域现象，并且还没有通过合理的方式解决（比如没设置Access-Control-Allow-Origin），那么本次请求的响应就会被浏览器忽略，同时抛出错误。
