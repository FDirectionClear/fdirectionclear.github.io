---
layout: post
title: 组件更新（上）
date: 2019-09-03
tag: Vue源码分析
---

### Vue的组件更新在什么时候？

答：

之前谈到了依赖收集和派发更新的过程，在派发更新的过程中，很可能触发了render
watcher的回调函数，render
watcher的回调函数就是vm._update方法，但是这里的vm._update的回调函数并不是第一次渲染，是针对更新时的重新渲染，其具体细节和初次渲染有所不同，很多情况下涉及到了经典的diff算法。

![](/images/posts/2019-09-03-VueSource-componentUpdate1/6361afd1d11e65f935caddd89d09211b.png)

### Vue实现组件更新的大体流程是什么？

答：

1.  首先执行了vm._update方法，进入patch方法，在patch方法中，Vue根据preVnode=vm._vnode的存在执行了不同的逻辑，如果preVnode不存在，说明是初次渲染，只需要按照之前文章中提到的挂载方式即可实现。但是对于preVnode存在的情况，说明本次更新并非是初次渲染。出于对性能考虑，在初次渲染的dom存在的情况下，直接大范围清除已经存在的dom，然后再重新渲染上去显然不是一个明智的方式，我们需要做的是对dom的更新部分作出判断，尽可能的缩小对dom的回流重构，这也正是vue能智能的选择出最快速的dom更新方式的原因。

2.  为了能采取最明智的更新方式，在patch函数内，vue再次判断了新旧vnode是否是明显相同的，如果不是明显相同（标签，是否注释，input类型，是否定义了data属性），说明本次dom的更新需要‘大手术’，直接找到需要更新位置的根元素，然后创建全新的dom挂载到根元素上，更新父的占位符节点，删除原来的旧dom节点即可，也就是经历了一个：

**创建新节点 -\> 更新父占位符节点 -\> 删除旧节点**

的过程。

如果新旧节点明显相同，就要按情况进行判断，如果新旧节点都是同一个子组件，那就要判断是不是子组件的props等属性需要更新，如果props等属性需要更新，进而触发子组件的更新过程。**如果新旧节点不是组件，那就要去判断他们的子元素是否相同，如果不同则进行智能的更新，这个地方就要涉及到diff算法。**

*TIP：*

*别忘了patch函数是负责调用createElm去根据vnode渲染dom并挂载到dom树上的根本方法。当存在oldVnode时，既然要挂载dom，patch函数就要能识别出oldVnode和vnode的差异，并选择合适的方式去更新dom。*

### 三、Vue是如何判断此次更新不是首次渲染的？

答：

首先我们需要知道一点，初次渲染时，vm._vnode是undefined，只有首次渲染过后vm._vnode才有值。

![](/images/posts/2019-09-03-VueSource-componentUpdate1/25f65d0c7fe969613638e3163f15a3c0.png)

另外我们还需要注意一点，vm.\$el是根元素，他是一个原生的dom节点并不是vnode，而preVnode是一个vnode类型的。

所以想要判断是不是首次渲染就很简单了，首先，我们只需要在_update中判断preVnode是否存在，如果存在说明不是首次渲染，是更新阶段，如果不存在，就说明是首次渲染。对于首次渲染和更新的patch逻辑区分是需要在patch函数内体现，所以在update阶段只需要一个选择一个合适的参数带入__patch__中，提供给patch函数区分逻辑的标志即可。

![](/images/posts/2019-09-03-VueSource-componentUpdate1/d1187e99101e846824578b05e19f8b65.png)

![](/images/posts/2019-09-03-VueSource-componentUpdate1/f89d010417acb08a2ae24930f1fcbd89.png)

### Vue是如何判断新旧节点是否相同的？

答：

Vue在判断本次patch是针对组件更新的时候，会先把oldVnode和vnode进行一次对比，对比的方式是通过sameVnode方法：

![](/images/posts/2019-09-03-VueSource-componentUpdate1/3a99e97786b5ee39c303f5e271073a48.png)

如果sameVnode为true说明新旧节点相同，如果oldVnode不是一个原生dom，那么就足以证明新旧节点相同，新旧节点相同就没有必要进行‘当前层’的dom更新了，更新的关注点应该在‘当前层’的子元素上，这时应该进入patchVnode进行子元素和子组件的更新处理，有必要时，还要进行diff运算。

sameVnode判断新旧节点是否相同，是根据新旧节点的vnode.tag是否相同、是否为注释节点、是否都定义了data属性，input的类型是否相同进行判断的，只有同时满足了以四个条件，才能说明新旧节点相同。

![](/images/posts/2019-09-03-VueSource-componentUpdate1/cb972e9db1987c71d970bb30351711ed.png)

### 对于新旧节点不同，Vue是怎么处理的？

答：

新旧节点不同，说明sameVnode方法返回false。应该进入else中的逻辑。

![](/images/posts/2019-09-03-VueSource-componentUpdate1/f1ac88c966b33b03c33d50bea265fe35.png)

之前也提到过，组件更新对于新旧节点不同主要经过以下四步：

1.  创建新节点（同时获取父节点）

2.  更新父的占位符节点

3.  将新节点插入父节点

4.  删除旧节点

**对于创建新节点的过程**：

![](/images/posts/2019-09-03-VueSource-componentUpdate1/71b1f483f75b92ad1152a2884d4c8962.png)

**对于更新父占位符的情况**，首先需要知道组件vnode实例的parent实例不会是渲染vnode，只能是组件vnode。以下面的情况为例：

![](/images/posts/2019-09-03-VueSource-componentUpdate1/c4c23fff088dd2d73d2492dcd2fb7023.png)

ul的vnode的parent是helloworld组件的组件vnode（tag是

![](/images/posts/2019-09-03-VueSource-componentUpdate1/9b8b225ac941120b5b6d59de1bd0d576.png)

），**而helloworld的vnode的parent是undefined，而不是div.App-container的渲染vnode，也不是顶层组件App的组件Vnode（因为组件更新的范围就是div.App-container，我想Vue这么设计很可能就是为了组件更新的时候容易限定更新范围）：**

![](/images/posts/2019-09-03-VueSource-componentUpdate1/e6bd8d4e107008ca946bcb059347d4a2.png)

因为更新的dom很可能出现在组件内部，更新占位符节点主要是针对该新节点所属于组件vnode的parnet的更新，和这个新节点渲染vnode的parent的更新。

*TIP：*

*什么是占位符？*

*答：*

>   *占位符主要是针对组件vnode而言，对于以下例子来说：*

![](/images/posts/2019-09-03-VueSource-componentUpdate1/5a80e92c8a0c10e2e89099c269dc6d55.png)

>   *组件的占位符可以表明某一坨dom是属于哪个组件的，他不会被渲染到页面上。因为一个dom集合如果没有特殊标记，那就很难知道这个dom集合是属于哪个组件的，并且组件的层级关系也很难通过一坨dom来表明，但是如果有了占位符，就能很方便的判断出当前这坨dom是属于哪个组件之下，反过来讲，占位符可以表示这坨dom属于哪个组件，对于嵌套层级比较深的render
>   vnode而言，他的children属性中只要存在某些组件的占位符即可，必要的时候我们可以通过这个占位符找到相关的dom或者当前组件的子组件。同时也为各种操作（比如这里对helloword和button进行diff算法运算就很方便）都提供了便利。我想这也就是组件vnode的存在价值。*

**对于将新节点插入父元素的过程**，createElm就已经做了，在这里不再赘述。

**最后，我们只需要把老的节点删除即可:**

![](/images/posts/2019-09-03-VueSource-componentUpdate1/7b5f17453117f1411de04697177778fd.png)
