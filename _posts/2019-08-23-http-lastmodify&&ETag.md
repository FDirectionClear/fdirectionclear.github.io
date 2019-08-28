---
layout: post
title: LastModify和ETag及时更新缓存问题的解决方案
date: 2019-08-28
tag: Http
---

之前的文章中说过，本地缓存虽然可以提高性能，但是却不能解决缓存及时更新的问题。如果我们希望浏览器能够判断服务端的文件是否改变，从而决定是否读取缓存文件，这就需要使用到缓存验证相关的设置。设置缓存验证，主要可以结合no-cache并设置Last-Modified或ETag两种方式。

### Last-Modified与If-Modified-Since

有一种策略是在请求的时候根据**服务端文件修改的时间**和**客户端请求的时间进行对比，从而来判断缓存是否可用的。**

LastModified的值是服务端设置文件修改时间，存在于消息头中，当然也不一定是时间，只不过最好使用时间，这样更符合规范。为了证明不使用时间也是可以的，我们在下面的实验不用时间戳去做验证。

If-Modified-Since的值正是客户端设置的，存在于请求头中，他的值正是上次客户端请求文件时，服务端对这个文件设置的LastModified的值。他的值如果与Last-Modified相同，说明文件没有被修改过，依旧是最新的，那么可以使用缓存。

*TIP：*

*在次强调，http头中的信息只是一种单纯的信息传达，他只是一个协议，而非强制性的。我们完全可以乱设置http头信息，然后在服务端或者客户端进行所希望的处理，或者反过来。虽然这么做是可以的，但是事实上这样做好不好还需要多说么？。。。*

### Etag

Etag相比于LastModified更加严格，他通过数据签名的方式进行文件的比对。所谓数据签名，就是指文件的内容，他会根据文件的内容计算出一个数据签名，只要文件内容得到一点点的修改，数据签名就会改变，如果不修改文件，那么数据签名就永远不会改变。最典型的做法是对内容进行hash计算。

Etag的使用需要配合客户端返回的if-Match或者if-Non-Match使用。和if-Modified-Since相似，他的值也是携带上一次服务端返回的Etag。

### Last-Modified和Etag的实验验证

1.  还是要利用上次创建好的server.js，缓存验证通常是在max-age时间较长的情况下需要的，我们先拉长max-age的时间，然后添加no-cache和Last-Modified和Etag。

![](/images/posts/2019-08-23-http-lastmodify&&ETag/4a0347d6b2f0459ca8dafc1edcdc50c4.png)

*TIP：*

*既然要缓存验证，就需要请求到服务端。所以要设置no-cache来阻止浏览器根据max-age直接读取本地缓存从而不发送验证请求。*

![](/images/posts/2019-08-23-http-lastmodify&&ETag/37a6f466e773f9e35dc6aea9a2cb7fca.png)

1.  什么都不做，直接刷新浏览器

![](/images/posts/2019-08-23-http-lastmodify&&ETag/1aafa81aa895bdffda707ead7a230caa.png)

>   需要注意的是，虽然if-Modified-Since和if-none-Match匹配上了，但是请求所返回的内容（script.js）还是得到了。**这说明了一点：Etag和last-Modified只是一个单纯的验证方式而已，并不会决定服务端或者客户端的处理逻辑，处理逻辑是我们根据响应报文的返回值自己在服务端编写的。这也充分体现了http协议是可以打破的，遵不遵守要看我们的‘职业道德’。**

![](/images/posts/2019-08-23-http-lastmodify&&ETag/f8cc41b40d10708d0d1513471e6489a3.png)

![](/images/posts/2019-08-23-http-lastmodify&&ETag/9af54f68172d6567eda8cf720a6aeacf.png)

>   3、修改服务端数据，并添加缓存验证的逻辑

![](/images/posts/2019-08-23-http-lastmodify&&ETag/f062b2b3fbf209f490ad5932213e0083.png)

**需要注意的是，我们应该返回304状态码，因为此时我们希望浏览器回去读取本地缓存，服务端并没有什么，同时也不应该有返回去的文件，否则缓存验证就无意义了。另外，即使是让浏览器读取缓存，我们也不要忘记应该返回对应的消息头！如果此处不设置消息头responseHeader就使用了默认值！**

5、重启服务器，连续刷新浏览器。

![](/images/posts/2019-08-23-http-lastmodify&&ETag/d20d737f13636b3a1347434f3457a4cb.png)

现客户端返回了304，浏览器看到304就去自动读取本地缓存了。

1.  改变服务端的etag，重新刷新浏览器。

![](/images/posts/2019-08-23-http-lastmodify&&ETag/c1b5e57dd161c6293522c93cbdf83e44.png)

我们依旧重启服务器，然后刷新浏览器，看看浏览器是否还是用777作为etag，如果是777，那么是否的打了新的数据script
loaded twice。

![](/images/posts/2019-08-23-http-lastmodify&&ETag/6b9bb76faf6b66972fead081dea6d426.png)

再次请求，发现状态码发生了改变。返回200，不再读取缓存。if-none-match也不再和etag匹配了。

![](/images/posts/2019-08-23-http-lastmodify&&ETag/1aafc32e7cf6ccd2a2de21538bfd1a27.png)

![](/images/posts/2019-08-23-http-lastmodify&&ETag/dd8f206da91532acad572ea74873ec33.png)
