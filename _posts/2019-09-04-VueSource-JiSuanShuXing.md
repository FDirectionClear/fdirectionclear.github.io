---
layout: post
title: computed计算属性
date: 2019-09-04
tag: Vue源码分析
---

computed计算属性
================

### computed的初始化时什么时候开始的？

答：

在执行new Vue时执行了vm._init()方法，在this._init中又执行了一系列初始化操作：

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/d55abe46897abb33f680881055948aff.png)

然后执行initState，在initProps，initMethods和initData后执行了initComputed。

### 二、为什么computed的初始化要在data和props之后进行？

之所以要initComputed在后，是因为computed中的‘缓存功能’，只有当computed所依赖的变量发生改变时才会重新computed。由于‘依赖的变量’发生变化时，要告知computed，所以‘依赖的变量’也必须是响应式的，因此initState、initData在先也就很好理解了。

### 三、computed是如何实现监听依赖变化然后重新算在触发模板更新的？

computed为了能感知到内部依赖的变化，就要监听内部依赖，这和render
watcher监视data是一个道理，因此Vue在实现computed时也是通过了观察者模式，让每个computed

都持有一个watcher，然后订阅内部依赖的响应是对象的dep。这样当内部的响应式对象发生改变就会触发dep.notify方法，从而触发computed
watcher的回调，这样就实现了对内部依赖数据监听的功能。

computed单单根据依赖的变化进行重计算是毫无意义的，computed的重计算需要伴随着dom的更新才有意义，因此render
watcher的任务就不仅仅是订阅props、data的dep了。对于computed的dep也要订阅。这也就需要computed属性也持有一个dep实例才行。

Vue巧妙的将computed
watcher设计成了一种可以持有dep实例的特殊watcher，一旦持有了dep实例，computed
watcher也就有了进行依赖收集和派发更新的能力。

所以大体上讲，computed的响应式实现是这样子的：

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/7f6401eee8c4da486b6cad67c89c66a2.png)

### computed的实现细节是什么样子的？

首先执行this._init，然后执行this.initState，然后执行initComputed。在initComputed中，主要做了四件事情：

1.  为vm.computedWatchers创建一个空对象，用来存放输入当前Vue组件实例的全部computed
    Watcher

2.  遍历computed配置项，然后获取computed函数。

3.  用computed函数创建computed watcher并保存在vm.computedWatcers中。

4.  用defineComputed为computed属性添加getter个setter，同时检查computed的名称是否在data和props中存在。

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/bcca6eda1715c1f7ad38e682cea304e8.png)

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/f7a188a1f9bf1ff08ec7d54ad395d79e.png)

最终创建的getter函数如下：

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/1c9c118d33090fab374066ee6adc9760.png)

由于在初始化computed watcher时带入了computed
watcher相关的配置，因此初始化computed watcher时会和初始化普通watcher有所不同。

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/8eb592472e6a05ad3854602c07469ba9.png)

由于带入了相关配置，this.computed为true。可以看到，computed
watcher也持有一个dep实例。需要注意的是，在Watcher构造函数中无论什么watcher都执行了watcher原型上的get方法，也就触发了pushTarget将Dep.target设置为当前的computed
watcher。

当模板编译时访问到相关computed时，会触发computed的getter：

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/1c9c118d33090fab374066ee6adc9760.png)

此时会进行computed watcher所持有的dep进行依赖收集：

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/c1a2b7b83495273b9a378598ddf4e11a.png)

依赖收集的过程之前已经提到过了，这里就不再提了。

然后执行watcher.evaluate()方法去进行求值。在求值的过程中，因为执行了get()方法，因此触发了computed函数，在执行函数的过程中，访问了各个依赖的响应式变量，响应式变量的getter被触发，也就进行了依赖收集将此时全局的computed
watcher塞入了自己subs中。此时首次渲染完成。

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/d3690e270d42e98145aaf925ebfb6855.png)

当依赖的数据发生变化时，由于触发了依赖数据的setter，因此会触发computed
watcher的update方法：

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/f277b622d00bf999fcb0af786bb01574.png)

getAndInvoke方法也很简单，就是检查新老值是否相同，然后执行回调函数。

![](/images/posts/2019-09-04-VueSource-JiSuanShuXing/8ed5c7ec6bc16a79bb9cb474cb74a233.png)

这样，订阅了computed watcher 的 dep 实例的render
watcher就得到触发，模板更新也就此完成。
