---
layout: post
title: 侦听属性
date: 2019-09-04
tag: Vue源码分析
---

侦听属性
========

### Watch侦听属性是什么时候进行初始化的？

答：

和computed一样，也是在initState中进行初始化的，在initComputed之后执行的。这个过程在created生命周期之前。

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/56310550ba00b92233c93ddad55d8c03.png)

### Watch侦听属性是如何进行初始化的？

答：

watch侦听属性的实现也是通过观察者模式实现的。watch侦听属性的watcher是一个user
watcher。

watcher的初始化也是通过遍历watch选项，对每个watch项都通过调用createWatcher方法去进一步执行vm.\$watch方法去创建user
watcher。

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/1c004d34a59ebd580422f08ba5dd3aa7.png)

值得注意的是，由于watch可以写成一个对象，也可以写成一个数组，也可以直接写成一个回调函数，以下是来自官网的写法示例：

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/ecedce5ca64dfa10940e5c565d00f3df.png)

所以，为了能够从各种数据结构的watch中获取回调函数，Vue单独设置了一个createWatcher方法去作为一个中间层去提取watch的回调。然后通过核心方法vm.\$watch去真正的创建use
watcher。

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/cd941e13cde6670557fb7c395a821e84.png)

**Vue之所以把vm.\$watch单独抽离出来，是因为Vue希望可以通过vm直接调用\$watch方法去动态的监听Vue的某些响应式对象。**

在vm.\$watch方法中，Vue首先初始化了user
watcher的配置，这里主要就是options.user设置为true，表示这是个user
watcher。然后实例化watcher。值得注意的是，如果watcher开启了immediate配置项，那么watch的回调函数就应该在创建use
watcher的时候立即执行，**从宏观角度上来看，这个回调正是在initWatch触发的时候，即created钩子执行前同步执行的。**

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/8c97377c35256383e844ace72bded0e8.png)

在初始化use watcher后，需要的就是让use watcher订阅watch那个对象的dep实例了。

这个依赖收集的过程，是在new Watch中触发的，我们先回顾一下Watcher的构造函数：

（1）初始化配置

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/e250c9c4e01d1984de63952f75e2cb6b.png)

（2）解析索要侦听的变量名

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/ce2cabcc72f7fccdbe85304c974c56fa.png)

（3）触发get进行依赖收集

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/38a655800f039f0008d2228da1598c89.png)

（4）getter中触发了依赖收集的过程

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/bc4c63b08b6e78660242b878b0fae8ad.png)

在执行get后，首先执行pushTarget函数，把当前正在计算的user
watcher赋值给Dep.target。之后执行getter，在执行getter的过程中，出发了vm[‘watch所监听的变量名’]，因此在此经过_proxy代理，直接访问到了vm上对应的的响应式对象。从而触发了响应式对象的getter，进行了依赖收集，将当前的user
watcher收集到自己的subs中。

至此user watcher的初始化过程也就到此结束。对于派发更新的过程基本上和render
watcher相似，只不过一个地方会存在区别，那就是sync配置项：

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/fe31e3cd3b3fef6b590719e15e576d95.png)

如果没有开启sync配置项，那user watcher就和render
watcher一样，先推入队列中，然后下一次事件循环里触发回调，也就是触发watcher.run()，从而触发方法基本一致。

### watch如何监听深层对象？

答：

可以开启配置项：deep: true，然后在初始化user
watcher时，进入get方法后会判断一下deep属性，如果deep开启会执行traverse方法。

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/16c1a0570b7021d962df6a281b586c1a.png)

traverse方法就是递归的去对深层对象进行依赖收集，将当前的user
watcher添加进深层子对象所持有dep实例的subs中，以此来实现触发子对象，也能触发父对象的watcher回调（严谨的说和父对象相同的watcher回调，并不是父对象的watcher回调）：

![](/images/posts/2019-09-04-VueSource-ZhenTingShuXing/5d27d6e657bb8250215e34c379000b89.png)
