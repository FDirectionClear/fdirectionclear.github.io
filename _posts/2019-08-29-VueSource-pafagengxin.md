---
layout: post
title: 派发更新
date: 2019-08-29
tag: Vue源码分析
---

### Vue是如何实现派发更新的？

答：

派发更新对应的是响应式对象的getter部分：

![](/images/posts/2019-08-29-VueSource-pafagengxin/31a50a929f2259125a9d8a6981541284.png)

在响应式对象被修改后会触发setter。我们可以注意到，在整个defineProperty设置getter和setter之前，Vue先缓存了这个数据对象的getter和setter，因为Vue希望用户自己设置的getter和setter能够比Vue所设置的getter和setter更优先。

![](/images/posts/2019-08-29-VueSource-pafagengxin/11beab6e2fc219622d083b9f2d2f3ef5.png)

在执行过getter后，Vue拿到了这个数据对象的新的值，经过新老值判断，如果新老值相同，则会退出set函数。**如果不相同则会优先调用用户事先定义的setter方法，然后对新值也通过observe方法为数据添加Observer，最后执行这个数据对象所持有的dep的notify方法去触发订阅者的回调函数。**这也正是setter的主要逻辑部分。

![](/images/posts/2019-08-29-VueSource-pafagengxin/2169ec4d7ef69b325ef3c48f637eadf1.png)

在触发dep.notify方法非常简单，就是对该数据对象所持有的dep中的subs的每一项进行遍历触发update方法，也就是触发了订阅了这个dep的所有watcher的update方法。

![](/images/posts/2019-08-29-VueSource-pafagengxin/6c37c3a5b26f0d9acb9d8c9b76e02f0d.png)

watcher的update方法也非常简单，在这里他判断了watcher类型，对于我们普通的render
watcher，这里就执行了queueWatcher方法，这个方法主要是想把同一时间内需要回调的watcher都推入一个队列中，防止多次执行或者重复执行，具体我们来看：

![](/images/posts/2019-08-29-VueSource-pafagengxin/c27804574b258e44cebb407704309196.png)

![](/images/posts/2019-08-29-VueSource-pafagengxin/1ce5da6e1f26c92b2da240779315a81e.png)

queuewatcher的执行逻辑也不复杂，我们需要注意的是queuewatcher方法被单独的封装在了一个模块中，他的方法体中借用了模块中的定义的顶级变量：

![](/images/posts/2019-08-29-VueSource-pafagengxin/29a0f8c7ea16f686408cebe46fb9f2c7.png)

queue是一个最核心的变量，他是一个有watcher组成的数组。在我们的开发中，经常会一次性改变多个响应式对象的值，因此可能会在一次逻辑的执行中，执行多次setter从而触发多次同一个watcher的update方法，从而触发watcher的回调，如果watcher是render
watcher，那多次触发就会多次进行渲染，可能在同一轮事件循环中，就有多个响应式对象需要触发render
watcher，对于模板中‘提到’的那些变量，我们没有必要每改变一个就要渲染一次，同一轮循环中，我们只需要以最终数据重新渲染一次就好。

为了实现这一点，我们就需要把同一轮事件循环中的所需要触发回调的所有watcher都推入一个队列queue中，需要多次触发的同一个watcher只添加一次就不再添加，然后在下一轮事件循环中统一触发回调，这样就能实现需求。

![](/images/posts/2019-08-29-VueSource-pafagengxin/0f927cb3f1dc5ee0739456e9ef52abaf.png)

还有一个很关键的核心变量就是waiting，我们可以从源码中看到，第一次进入queuewatcher方法后就会把waiting从false置为true。因为waiting是模块中的顶层变量，所以下次进入就不会执行flushSchedulerQueue方法了。

flushSchedulerQueue方法是用来遍历执行queue中的watcher回调的方法。因此执行一次即可。

![](/images/posts/2019-08-29-VueSource-pafagengxin/4779fd31ef031a94706397f734e2c7f2.png)

![](/images/posts/2019-08-29-VueSource-pafagengxin/d5edeb452c0527b3e39017c26a2271fd.png)

在flushSchedulerQueue方法中，主要处理了三件事情：

1.  队列排序

>   之所以要排序，是因为watcher是按照程序执行顺序推入queue的，但这并不代表，watcher的回调顺序也应该按照添加顺序进行。因为一些watcher的回调如果后进行，很可能之前已经执行过的watcher回调就白执行了。

>   排列具体解决一下三种问题：

1.  组件的更新由父到子，因为父组件的创建是先与子的，所以watcher的创建也是先父后子的，执行顺序也因该保持先父后子。

2.  用户自定义的watcher要优先渲染watcher执行。

3.  如果一个组件在父组件的 watcher 执行期间被销毁，那么它对应的 watcher 执行都可以被跳过，所以父组件的 watcher 应该先执行。

4.  队列遍历

>   队列遍历主要就是对排序后的queue进行遍历，按顺序触发每一个watcher的回调。主要是watcher实例的run方法。

![](/images/posts/2019-08-29-VueSource-pafagengxin/5cb2f84ab3b48d3039315f8481434341.png)

>   run方法实际上就是调用了相关watcher的回调函数，对于render函数而言就是调用的这段逻辑去进行组件的重新渲染，但是渲染方式会和初次渲染不太一样：

![](/images/posts/2019-08-29-VueSource-pafagengxin/7390099aa34492ff0f5afc240d64f4fc.png)

1.  状态恢复

![](/images/posts/2019-08-29-VueSource-pafagengxin/5544954990fc487619b6f2a0bc3d759d.png)

状态回复就是这个部分，很简单，就是清空队列，把之前的顶级变量重置。表明一次完整的派发更新过程已经完成，期待下次派发更新。

至此，派发更新的过程就算是结束了。
