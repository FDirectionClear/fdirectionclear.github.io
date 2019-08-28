---
layout: post
title: CORS与复杂跨域请求
date: 2019-08-28
tag: Http
---

本篇笔记是针对之前的笔记中记录关于CORS设置跨域请求的一些细节的补充。通过服务端设置响应头属性Access-Control-Allow-Origin的方式可以指定接受跨域请求的域名。但是仅仅在responseHeader中设置这一种属性是不能解决复杂请求的跨域问题的。**本文就着重对一些复杂请求的CORS跨域解决办法进行记录。**

### 什么是简单\|复杂请求

只要同时满足一下两大条件，就属于**简单请求**：

1.  请求方式为一下三种之一：

    1.  GET

    2.  POST

    3.  HEAD

2.  HTTP的头信息中不超过一下几种字段

    1.  Accept

    2.  Accept-Language

    3.  Content-Language

    4.  Content-type ： 只限三个值：
        application/x-www-urlencode、multipart/form-data、text/plain

    5.  Last-Event-ID

如果请求能同时满足以上条件，就属于**简单请求**，否则为**复杂请求**。

### 为什么谈及简单\|复杂请求

对于简单请求而言，上文说到的方式就能完美解决跨域问题，但是如果是复杂请求，单单一个Access-Control-Allow-Origin就不够了。

先来看一个例子：

![](/images/posts/2019-08-28-http-complexCors/5fb855b03fff79457a92c814fa907bc0.png)

![](/images/posts/2019-08-28-http-complexCors/c05868a4e08ab69c467191eedb97edba.png)

创建好用于提供index.html的服务器后，创建提供跨域服务的server2.js

![](/images/posts/2019-08-28-http-complexCors/e964a39f3f00782a8a15bac3e48b71fd.png)

然后访问localhost:8888

![](/images/posts/2019-08-28-http-complexCors/e61e475f2dcbcb8b538cfe56c2bee203.png)

发现依旧出现了跨域拦截的问题，我们打开控制台，进入Network查看请求情况：

![](/images/posts/2019-08-28-http-complexCors/f6a6268c2e525a90e6294bb1aef3133a.png)

这里倒数第一个localhost是向8888请求得到index.html，我们不必管他。

只关注倒数第二个localhost即可。

注意到，这个请求的方式是OPTIONS，**是一个预先用来询问服务器的请求方式，也叫做预请求**。在他的Request
Headers携带了两个属性，分别是：

**Access-Control-Request-Header和**

**Access-Control-Request-Method。**

ACR-Header中包含了我们自定义的头信息中多出的属性。ACR-M中则携带了我们的请求方式。

回顾开篇所说，区分复杂的请求和简单的请求应同时关注请求的方式和请求头中的一些关键字段。浏览器对于复杂的跨域请求大致会经过以下流程：

（1）首先，我们的在server2.js中设置了Access-Control-Allow-Origin :
“\*”，因此浏览器知道了这是一个可能具有跨域资格的请求。

（2）之后，浏览器发现，这是一个复杂请求，**于是为了判断这个复杂请求最终是否被允许跨域，他需要先预先询问服务器**，你都允许什么样的复杂请求。于是发出了预请求OPTIONS

（3）在OPTIONS之后，浏览器就知道这个请求到底具不具备跨域资格，如果符合资格那就正常返回服务器的值，如果不符合资格就直接忽略服务器的响应，同时抛出一个错误。

### 服务端针对复杂请求作出响应

其实很简单，我们只需要向浏览器返回认可跨域的属性和方法即可。

![](/images/posts/2019-08-28-http-complexCors/b0f265af1dcfcf9bb57112199ecf6230.png)

这样，我们修改index.html中请求方法为PUT，同时依旧保留自定义头信息X-Test-Cors：‘456’。

![](/images/posts/2019-08-28-http-complexCors/9ced2a80b009de9eee3ac580c3a1b6f1.png)

这样我们就可以正常的实现请求了。

![](/images/posts/2019-08-28-http-complexCors/b3432c295090a1b23fd0b83e05704c51.png)

### ？？？遗留问题：

**事实证明浏览器的预请求是在浏览器已经判断该请求可能具备跨域资格的前提下进行的，也就是说，在server2.js中是设置了Access-Control-Allow-Origin
:
“\*”，如果没有设置，就直接阻止并报错了，这里正因为设置了，浏览器才没直接阻止跨域请求，进而发送了预请求去进一步判断跨域资格。那浏览器在预请求前是如何知道server2.js中是否设置了Access-Control-Allow-Origin
: “\*”的呢？**

**如果和简单的跨域请求一样，先得到服务器的数据，再根据头信息判断是否可以跨域，那为什么复杂请求就不能类似的进行一次性判断呢？反而要经历一次预请求？**

#### 鸣谢链接：

-   阮一峰老师博客：<http://www.ruanyifeng.com/blog/2016/04/cors.html>
