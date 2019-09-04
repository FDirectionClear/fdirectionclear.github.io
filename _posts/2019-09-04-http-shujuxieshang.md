---
layout: post
title: 数据协商
date: 2019-09-04
tag: Http
---

我们都知道，在http请求的请求头和消息头中会有一些值为MIME类型或者语言类型zh-CN等...

设想一下两个人互相写信的情况，你希望另一个寄来一张照片，但是如果在信中不提及‘你应该把照片寄过来’这样的信息，那么另一个人是不容易知道你想要什么样的邮件。如果想要让回信方明确自己想要的是什么类型的回信，那就要在信中说好。

在http请求中，发送的数据类型，和期望接受的数据类型有太多种，客户端为了能表示自己所期望的数据类型和一些其他的信息，就不得不在请求报文中提到。

![](/images/posts/2019-09-04-http-shujuxieshang/7e85521779e06bc15fb0ae9bc6c6edfd.png)

### 数据协商头信息

1.  请求

2.  Accept：声明客户端所期望的数据类型

![](/images/posts/2019-09-04-http-shujuxieshang/cf1758714e11a5338dcd21349ff05d67.png)

1.  Accept-Encoding 客户端所期望的数据压缩方式

![](/images/posts/2019-09-04-http-shujuxieshang/a51adb968979c1ef464f2f5420bc6716.png)

1.  Accept-Language 客户端所期望返回的语言类型（国内一般是中文，国外一般是英文）

![](/images/posts/2019-09-04-http-shujuxieshang/99a510ac3fa8d40dd43f78359a4c1201.png)

值得注意，语言同样可以接受好几种类型，后面的q的值表示每种语言返回的权重，权重越高期望就越高。这里我们优先返回简体中文，其次返回英文，再其台湾的繁体中文，之后就是日语和韩语。

1.  User-Agent：浏览器信息，一般表明发起请求的浏览器是移动端的还是PC端，浏览器的内核是什么，发送请求的操作系统是什么.....服务端可以根据这个头信息来返回不同端的网页。

2.  响应

3.  Content-type: 与Accept相对应，是服务端返回的真实数据类型

4.  Content-Encoding: 与Accept-Encoding相对应，返回的数据压缩方式

5.  Content-Language：与Accept-Language相对应，返回的语言类型。
