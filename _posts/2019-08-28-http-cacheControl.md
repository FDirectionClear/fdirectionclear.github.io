---
layout: post
title: CacheControl的含义和使用
date: 2019-08-28
tag: Http
---

### public & private & no-cache

众所周知，在http请求返回的过程中会经过我们的浏览器，以及众多的代理服务器。

-   public的意思也就是在任何一个节点都可以对数据进行缓存，缓存可以来自于浏览器自身，也可以是其中的某个代理服务器。

-   private：就仅限于在我们的浏览器使用缓存。

-   no-cache:
    可以在本地或代理服务器使用缓存，但是需要去原服务器端进行验证，在原服务验证未过期后，继续使用缓存。

### 到期时间

-   max-age = \<seconds\>
    max-age是专门为我们浏览器所设置的在max-age所设置的时间内，资源不会被浏览器重新请求，而是直接使用本地缓存。

-   s-maxage = \<seconds\>
    和max-age的功能是类似的，不同点在于s-maxage是对代理服务器而言的，而max-age是对浏览器而言的，**当我们同时设置了s-maxage，和max-age，浏览器会先根据max-age来判断是否应该发起请求，在发起请求后，s-maxage就会被服务器读取，在s-maxage的过期时间内，代理服务器就会直接返回缓存中的数据**。

-   max-stale : 即使max-age过期，在max-stale的时间内，浏览器还是会使用缓存。

### 重新验证

Must-revalidate:
对浏览器而言，在max-age到期之后，必须去**原服务器再起请求数据。**

**Proxy-revalidate：对缓存服务器而言，在s-maxage到期之后，必须去原服务器再次请求数据。**

**No-store: 注意和no-cache的区别，no-store是永远不使用缓存。**

**No-transform:
有一些proxy服务器会改变资源，比如他认为我们需要请求的数据太大了，就自动的帮忙压缩，no-transform的含义就是告诉proxy服务器，不要进行额外的处理。**

### cache-control: max-age实验现场

1.  **先起一个简单的服务器**

![](/images/posts/2019-08-28-http-cacheControl/0026e502d06f41a45509b7667152c4d2.png)

1.  在index.js中访问/script.js，同时返回脚本

![](/images/posts/2019-08-28-http-cacheControl/7c3784ed6f45248622cd64984351a406.png)

1.  访问8888，查看network

![](/images/posts/2019-08-28-http-cacheControl/205eff608b51c7c6ecab167b99a4a2bc.png)

![](/images/posts/2019-08-28-http-cacheControl/87b955d8efe48f855112e62d2cd01baf.png)

![](/images/posts/2019-08-28-http-cacheControl/55a8a5df78f643fc8c6577fc0bd296f2.png)

可以看到node服务器成功的返回了脚本，但是多次刷新浏览器后，我们依旧可以看到同样的信息，返回的状态码仍为200，且依旧是通过请求的方式，而不是from
memory cache。

1.  设置max-age

![](/images/posts/2019-08-28-http-cacheControl/ab77a9f06e56964f4a4360ad30b6db3b.png)

设置max-age为20，在20s之内，缓存有效。

首先初次请求

![](/images/posts/2019-08-28-http-cacheControl/f5438d31b59d81027b8afbe5b9257daf.png)

在20s内立刻刷新页面，可见此次script.js请求根本就没发送到服务端，直接from memory
cache且状态码为200（。**值得注意的是，Time为0，说明了资源拿取的效率可能在ms级以下了，好处不言而喻!**

*TIP：*

*因为没有经过服务端，所以浏览器在从缓存中得到资源后自动返回200，而不是304!状态码304相当于服务端告知浏览器去读取缓存，浏览器不会因为资源是从缓存中读取的就自动返回304！）*

![](/images/posts/2019-08-28-http-cacheControl/a34fa6021c44705adb8d4d4a882810aa.png)

20s后，再次刷新页面，可以发现，资源又被重新请求：

![](/images/posts/2019-08-28-http-cacheControl/76a8b3501104ecc7173547c87df72aa9.png)

但是这也暴露出了一个缺点：服务端的返回的内容改变，但是还在缓存时间内，浏览器依旧会使用本地的缓存资源。他生不知资源已经过期了。

为了验证这一点，我们首先延长过期时间至2000s，然后重启服务器，并刷新浏览器，获取第一次的请求资源后，改变node返回的js脚本内容（多一个’twice’字样）。此时不刷新浏览器也不清除浏览器的本地缓存，重启服务器后直接刷新浏览器。

![](/images/posts/2019-08-28-http-cacheControl/ebb19eb5611aa36e3442810554217b9c.png)

可以发现此次的浏览器依旧读取的缓存：

![](/images/posts/2019-08-28-http-cacheControl/758027894396f00ae9f428805c0a8ea1.png)

在打印的内容中也可以看出，浏览器依旧使用了过期的本地缓存：

![](/images/posts/2019-08-28-http-cacheControl/224f095a05e0f87914f34d4d7ff2f4cd.png)
