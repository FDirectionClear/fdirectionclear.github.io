---
layout: post
title: SplitChunksPlugin配置参数详解
date: 2019-09-04
tag: webpack
---

SplitChunksPlugin配置参数详解
=============================

### SplitChunks的默认配置

以下是常见的配置方式（按照官网的写法写了一遍）

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/8312bf5d500539d79e23a4fd8e312add.png)

### SplitChunks.chunks

常用的取值有三个”async”（异步）,“all”（同步和异步）,”initial”（同步）。

什么意思？异步就是只支持对异步引入的代码进行代码分割。同步的就是只对同步的代码进行代码分割。all就是无论怎样都进行代码分割。”async”是默认值，所以这也就是为什么在我们异步使用import()进行代码分割的时候可以不配置optimization的原因。

但是这里有一个点需要注意：

当我们设置splitChunks.cacheGroups.vendors为false的时候，无论怎样都不进行代码分割。

（具体稍后解释）

### splitChunks.cacheGroups.vendors

首先需要说的是，尽管在chunks选项中设置了针对什么方式引入的文件进行分割，但是并不代表所有符合chunks设置的方式的文件都会被分割。

*额外补充修改*：

*经过试验总结如下：*

1.  *如果是同步引入的文件，想要分割还得看这个文件是否符合cacheGroups中某一个“组”的条件。如果这个文件不符合任何一个“组”的条件，尽管引用方式符合chunks设置的引入方式（inital,
    all），也还是不会进行分割会进行代码分割的。对于同步引入的文件而言，只有同时满足分割条件，且有‘组’能接受，才会被分割。*

*对于这个现象，我个人的理解如下：*

*因为webpack要作出代码分割之后在index.html中引入，而对于没有‘组’的文件，统统属于可接受频繁变化的，也就是非静态的，让这些非静态的文件统一打包在main.js中，这样main.js中的代码就都是非静态的，一者变化就全变化即可，总之不会对其他大型静态文件产生波及。（这里说的非静态，就是指没有cacheGroups可以进入，也就是说需要接受频繁的更新）。*

1.  *如果文件是异步动态引入的，无论怎样都会进行分割（即使group设置为inital，这似乎是Webpack现版本的bug）。如果满足分组条件，那么就按‘组’打包，现象和符合‘组’条件的同步文件是一样的。但是如果不满足任何一个‘组’的条件，webpack依旧会对他们进行打包，在没有魔法注释的情况下，打包出的chunk名默认为0,1,2,3，.......。*

>   *对于这种现象，我的理解是：*

>   *异步引入的文件终究是一开始不会使用的，一定要在等到某个时机后插入head中的*

*script标签内。因为是异步的存在，所以异步文件中的代码无法在最开始就被整合在一个文件中引入index.html中，所以只能额外进行分割。*

*举个例子：*

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/4c58587e51cebc3bbd6a27cb6296be22.png)

*在这里动态引入了number.js，其中导出一个叫fangxiangming的方法，*

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/c6aa089e665d6885b0264dcec9c3553c.png)

*尽管Import看上去和同步的没什么区别，是在文件一开始就会执行，但是他还是异步的，终究要等到js的一轮事件循环后，请求number.js完成后，才会被插入到head标签中。*

*由于main.js中的代码是同步代码，是在一开始就插入到index.html中的，所以Number.js中的代码是无法被提前融合到main.js中的。既然不能融合，那就不得不将number.js分割出来，在适当的时机引入。而且，对异步的js文件而言，也无法找到不分割出单独文件，最后还能异步插入head中的办法。这也许就是无论怎么配置chunks，且chunks默认是async的原因。（这里有可能会想，既然无法用src插入，那就把代码直接吧需要异步执行的js代码插入进去scrpit标签中不就行了。但是如果这样的话，main.js就会变大，代码分割和没分割又有什么区别？）。*

看下面这个vendors组的默认条件：

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/5a652375dcd9a64c34e74be68ceefcf5.png)

这句话的意思就是说，当引入的文件是来自node_modules时，进行代码分割。

以我们之前所说的lodash为例，lodash是通过npm下载的，所以一定在node_modules下，因此符合vendors组的条件，就会生成默认命名为vendors\~xxx.[ext]的文件。

比如：

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/a7a3ac6305af5050a41a17109ac0e796.png)

如果我们设置vendors为”false”，那么也就是说，没有任何条件符合vendors组的条件，也就不会生成vendors组下的分割文件。

在这里我们还可以设置对应组内分割出文件的名称：

我们只需要增加一个filename即可。

比如

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/a78434ed7c427ed622acd44a26be3027.png)

*TIP：*

*使用filename命名会出现一个错误，目前不知道如何解决：*

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/06477fe698adbcc3e96082081bdd746e.png)

*怀疑是因为这一点：*

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/7951e7274cdf8e13fa706141c50b846c.png)

*意思是说防止使用全集设置filename，并且如果你没设置’inital’，webpack就会抛出一个错误。*

*但是发现将filename替换为name就可以正常打包，尚不清楚为什么？*

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/fbeb9e5dc60e7a264f1f1ae082641827.png)

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/2b84295bf69c78770a558995ec1fff98.png)

*额外注意：*

*由于已经指定了vendors组的name，所以魔法注释不再必要（但是推荐还是写上，因为这样可以声明当前模块的ChunkName），魔法注释是指单个模块的模块名，比如这里lodash的文件名为lodash，但是lodash会和其他满足条件的分割代码一同合并至vendors组下（如果vendors设置了filename，如果没设置则把vendors组下代码的单独分割。这时候我们的魔法注释就排上用场了），所以只需要指定一个vendors组名即可。*

### 多情况说明

在为cacheGroups设置了filename或者name（也就是组名）的时候，如果多个分割文件都符合某个组的条件，**那么这些文件就会被打包到一个文件中（如果没设置则还是把vendors组下代码的单独分割）**。这个文件的名就是name设置的名。**如果一个文件符合多个组的条件，那么就会根据每个组的权值而定，文件会进入权值高的组中，权值是由每个组中的proirty设置的，数值越大的权值越大。**

注意，设置chunks为initial，对于异步import的代码但是还是会分割（如果设置为async，那么对于同步的就不会作出任何分割，以下言论是针对chunks为inital，现在chunks：async是异步的），**而不是不满足‘任何’组额条件的就不进行分割，分割的方式就是以chunkName命名文件（来自魔法注释），只不过是不满足进入cacheGroup而已。从这一点就可以看出，cacheGroup上面的众多配置条件都是判断引入的文件是否应该进入cacheGroup组的条件而已。而不满足上面条件的才是不会进行分割的。**对于不符合任何一个cacheGroup组条件的，只进行默认的分割，以自身ChunkName命名分割的文件，他们不属于任何组，**如果没有通过魔法注释配置chunkName，就会以0,1,2,3...这样的命名。**

**看以下例子：**

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/5586899d77a20c124c271df98e9aa5ba.png)

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/e7e39cec997ef8a69a964b6eab2bc898.png)

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/fec049d470596cceba20d952a222fc7c.png)

**经过试验：filename可以使用placeholder，而name不可以使用。但是filename必须设置chunk:”initial”，否则打包会报错。（经过试验，即使设置为initial也会分割动态引入的文件，不清楚为什么，感觉这地方坑好多啊，和网友探讨说这是webpack的一个现存Bug，探究完后回来补充修改）。**

**问题得到解决：至于为什么只能是initial下才可以配置filename，这是webpack的一个bug，可能会在最新的版本中得到修复。。。。。。**

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/533918d1d685378f82f1e7525d156174.png)

splitChunks.minSize / splitChunks.maxSize
-----------------------------------------

这个条件就显然易见了，意思就是说如果引入的文件的大小符合maxSize和minSize就进行代码分割，否则也不进行代码分割。

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/3d8e5eb5996497471e4fc798f0c28247.png)

单位是Byte

一般设置minSize:30000(30KB)即可。

maxSize这个东西可配可不配，比如设置maxSize为50000，那么当一个文件的大小超过了50000的时候就会尝试二次拆分，如果文件不能二次拆分。那就会忽视这个参数。

maxSize的使用相对较少。

splitChunks.minChunks
---------------------

这个选项的意思是，一个文件最少使用几次才进行分割。在这里我们设置的minChunks是1，因此，只要lodash被**所有chunks**引入过一次，就会进行代码分割。如果将minChunks改为2，lodash只被引用了一次，所以就不会进行分割了。但是通常情况下都是1。

要说清这个概念，就不得不说说什么是chunks了，其实在每一次打包过后，生成的每一个打包好的文件都是一个chunks。

![](/images/posts/2019-09-04-webpack-×webpack_SplitChunksPlugin/e6d8f23221f5ed0712467835a9304ffd.png)

minChunks事实上就是指某个模块至少被多少个chunks所引用。在这里如果设置minChunks为2，那么只有当lodash被至少两个chunks所引用的时候才会打包。

需要注意的是index.html不是chunks，vendors.js就是lodash本身，main.js引用了lodash,所以一旦改成2，就不会对lodash进行分割了。

剩下的配置
----------

剩下的配置不多且不常用，通常情况下只需要默认配置即可，详见

Webpack4视频 4-6的7分钟左右。
