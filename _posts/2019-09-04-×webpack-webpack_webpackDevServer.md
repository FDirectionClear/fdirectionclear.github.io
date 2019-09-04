---
layout: post
title: WebpackDevServer实现请求转发
date: 2019-09-04
tag: webpack
---

WebpackDevServer实现请求转发
============================

先来看一个例子：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/be7131130e6e32076b4a0bcbf3dc739f.png)

这是一个React组件，我们使用了axios发送了一个ajax请求。

这个请求的接口是一个json文件，因为服务器那边设置了允许跨域发送请求，所以不必纠结为什么跨域还能请求得到数据的问题。

打开控制台我们可以看到返回的结果：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/ce3f11d294f02b24a584f6062a980cf8.png)

现在考虑一下开发环境的问题，因为在实际开发中，我们请求的后端接口可能只是用于测试的接口，在打包上线的时候需要请求线上的接口，**说白了，就是我们的ajax请求的接口可能随时更换**，因此我们通常使用一个相对url来充当万能的接口，就像这样：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/05d1cc50b47f3d83125f4b01bb7a31fe.png)

但是问题出现了！：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/2b17313cedb68cc6f6d725c1c666270e.png)

因为使用了相对的文件路径，没有指明域名，我们在本地测试的时候，域名就默认指向了

Localhost:8080，可是我们的数据在www.dell-lee.com上，这样就没法请求得到数据了。

为了能够再本地使用/react/api/header.json来作为接口，而不被默认填充域名localhost:8080，我们就得实现接口转发，也就是说，当我们在代码中请求以/react/api/开头的地址时，就要为这个相对的url默认填充一个固定的域名从而变成一个完整的接口路径。

这就是接口转发，很多插件都实现了这个功能，webpack也为我们提供了这个功能，使用起来非常简单：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/7bdb640f477d1d167a1bd3eb131fb3b8.png)

然后我们就可以在项目中使用/react/api/xxx来请求了，如果我们的接口域名发生变化，我们只需要修改这个配置所对应的域名即可。然后项目中所有以/react/api/路径开头的请求都会自动填充成以这个域名为开头的完整域名。

配置完成后，我们在重新打包尝试一下：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/0e4fe563da947e3e2b006e101bb02782.png)

可以看到已经成功的拿到了www.dell-lee.com下的数据。

当然可能发生改变的不仅仅可能是域名，还可能是路径中的内容：

只需要进一步配置即可：

![](/images/posts/2019-09-04-×webpack-webpack_webpackDevServer/78d652fb3ca4112028990af2b780ef6d.png)
