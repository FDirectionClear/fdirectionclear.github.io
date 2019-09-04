---
layout: post
title: 组件更新（中）
date: 2019-09-04
tag: Vue源码分析
---

### 对于新旧节点相同的情况是如何更新组件的？

答：

对于新旧节点相同的情况，我们是不需要对当前节点作出更新的，我们的关注点应该是他们的子节点。在经过sameVnode的判断后，进入patchVnode方法去更新当前节点的子节点。

![](/images/posts/2019-09-04-VueSource-componentUpdate2/6511058656100ce584fb2a1bfbdbb8ac.png)

当前需要更新的父节点的种类大体上分两种，一种的普通的vnode，这类vnode一般就是普通的dom节点。另一种是组件vnode。

对于普通的vnode父节点来说，我们需要做的是对他的子节点进行更新。他的子节点大概存在这4种情况：

1.  oldCh 与 ch 都存在且不相同时，使用 updateChildren 函数来更新子节点，涉及到diff算法，这个后面重点讲。

2.  如果只有 ch 存在，表示旧节点不需要了。如果旧的节点是文本节点则先将节点的文本清除，然后通过 addVnodes 将 ch 批量插入到新节点 elm 下。

3.  如果只有 oldCh 存在，表示更新的是空节点，则需要将旧的节点通过 removeVnodes 全部清除。

4.  当只有旧节点是文本节点的时候，则清除其节点文本内容。

由于2 3 4种情况比较好理解并且也没有比较复杂的逻辑，在这里就不再做出分析了。

我们主要针对第1种情况——当前需要更新的节点下，新老子节点都存在且不相同时的情况进行分析。

在这里先提前说明一下updateChildren方法的作用，他的主要功能就是借助diff算法为当前需要更新的父节点下的所有子节点进行一次性的更新，他接受5个参数，最重要的是前三个，分别是elm，oldCh，ch，分别代表当前需要更新父节点的vnode（也就是这个普通vnode）对应的真实dom节点，旧父节点的vnode的children，新父节点的vnode的children。因为组件vnode没有children，所以updateChildren方法的主要服务对象是普通的vnode。

![](/images/posts/2019-09-04-VueSource-componentUpdate2/c842a035320a2223208df7f8faceed5e.png)

对于组件vnode而言，他需要做的是更新组件的传值情况，比如组件的props就会在这里得到更新，如果props的改变触发了组件的重新渲染，那组件的render
watcher回调就会在下一个事件循环中得到执行，执行的逻辑和我们现在提到的是一样的。为组件更新props等属性的方法是prepatch方法，这个方法只存在于组件vnode中。

![](/images/posts/2019-09-04-VueSource-componentUpdate2/418c87bb9f7399baaaaa8dc19ced9054.png)

![](/images/posts/2019-09-04-VueSource-componentUpdate2/70614b142a655af6d98f81529c21b0b6.png)

组件类型vnode的更新最终必然会演变为普通类型vnode节点的更新。因此组件类型的vnode更新主要逻辑到此结束。

**现在，我们来看一下Vue更新子节点的精髓——diff算法，也就是我们的updateChildren方法。**

**diff算法的代码逻辑十分复杂，直接看源码不太合适，这里借用网上找来的一些相关图文来进行解释。**

**首先来看一个例子：**

![](/images/posts/2019-09-04-VueSource-componentUpdate2/7a9d3b6f677d5e99119b1e0d8dab4cdf.png)

在上面这个例子中，浏览器上原本显示的文案是A B C
D，在点击按钮后执行了reverse和push的逻辑，文案变成了D C B A E。

TIP：

>   Vue中使用方法对数组的操作，类似于reverse和push是会触发派发更新的。这是因为Vue重写了数组的相关方法。之后我会在写相关博文来记录。

为了能说清diff算法处理上面例子的思路，我们用图片进行描述：

首先updateChildren引入了四个指针，分别是oldStartIndex、oldEndIndex、newStartIndex、newEndIndex。分别指向旧节点下所有子节点的首位和新节点下所有子节点的首位，然后移动指针逐一对比old和new上的节点，如果有一对指针下的节点相同（sameVnode），那就对这个节点进行移动，移动到新节点所对应的最近的索引处。在移动后，将该节点所对应的两个指针进行移动，end指针向前，start指针向后，表示当前节点已经处理完成。当出现end指针在start指针前时，说明所有需要移动的节点都已经处理完毕，接下来只需要对比一下新老节点剩余的节点，对新节点有但老节点没有的节点，直接插入相应位置，对于新节点没有但是老节点有的节点直接删除即可。

![](/images/posts/2019-09-04-VueSource-componentUpdate2/3eddfd7767bd204fc2bc8a5a3c6481f4.png)

![](/images/posts/2019-09-04-VueSource-componentUpdate2/44614317df99d765f9a2ccb260913a8a.png)

![](/images/posts/2019-09-04-VueSource-componentUpdate2/9a8198fe9e9db78d4d5700184b49aa05.png)

![](/images/posts/2019-09-04-VueSource-componentUpdate2/2a2b0e32a90c776cb3c81138822e3f31.png)

关于diff算法的好的博文：

<https://segmentfault.com/a/1190000008782928>

![](/images/posts/2019-09-04-VueSource-componentUpdate2/fe31e507422add872c81d9c0064bd34a.png)

![](/images/posts/2019-09-04-VueSource-componentUpdate2/4afe445bf437590cc41ecbd36b2e0ea7.png)

### diff算法中为什么比较只会在同一层进行，不会跨层比较？

答：

要说清这一点。我写了一个小demo来辅助解释：

注意：下面的解释是依旧掺杂了大量的我的理解，但是还没来得及验证，仅供参考。

![](/images/posts/2019-09-04-VueSource-componentUpdate2/bf6c31332818f14c9864d593c847c142.png)

现在这里有一个app组件和一个helloword组件，helloword组件是app组件的子组件。flag为true，点击按钮toggle，会让flag取反，当flag发生改变的时候helloword内部的dom会发生更新。但是App下的dom并不会发生结构上的更新。

1.  **点击按钮**

>   flag变换为false，由于flag是响应式对象，并且被模板所依赖（进行了依赖收集），触发了派发更新，从而执行了App组件的render
>   watcher的_update回调，_update回调执行前，render()函数为我们获取了最新的vnode，根节点是div.App-contaienr。

>   未完待续.....
