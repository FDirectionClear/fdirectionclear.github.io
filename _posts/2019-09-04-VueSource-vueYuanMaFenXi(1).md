---
layout: post
title: Vue源码分析（一）从入口开始
date: 2019-09-04
tag: Vue源码分析
---

Vue源码分析（一）从入口开始
===========================

这一部分主要研究了import Vue from vue 的过程：

Import Vue from
vue这一行代码无非就是引入Vue模块，而这个模块正是Vue这个全局对象。

Vue的入口是在这里：

src/platforms/web/entry-runtime-with-compiler.js

在这个文件中，我们发现又引入了Vue

![](/images/posts/2019-09-04-VueSource-vueYuanMaFenXi(1)/5c59de7f3d556bdbbd3169583a4a9caa.png)

说明Vue这个全局对象的初始化并不是一步到位，而是多个文件配合组成的。

在当前的文件中，是为Vue绑定了一些诸如Vue.prototype.\$mount、\$options等继承方法和属性。

然后进入./runtime/index:

然后又会发现这个文件还不是Vue的最初配置，在这个文件中又引入了Vue

![](/images/posts/2019-09-04-VueSource-vueYuanMaFenXi(1)/d290a2fbb12fa4022348988a27548c07.png)

在当前的文件中是给Vue定义了一些全局配置和一些Vue.prototype上的方法。只是为Vue这个对象做一些扩展，不是很重要，可以先不要。

最关键的是这里：import Vue from 'core/index'

在src/core/index.js中：

在这个js文件中，依旧不是import Vue的终点：

![](/images/posts/2019-09-04-VueSource-vueYuanMaFenXi(1)/6baa029d15509b6c142c728cbdb63b1c.png)

他初始化了一些全局的静态方法

![](/images/posts/2019-09-04-VueSource-vueYuanMaFenXi(1)/6d3729dc63e1ec360f6c7c94962dbc7d.png)

同时为Vue.prototype上又定义了一些访问器属性。重点是他在这里初始化了一些全局的静态方法。

进入./instance/index，这里才是Vue初始化的终点。

![](/images/posts/2019-09-04-VueSource-vueYuanMaFenXi(1)/fe8ec240901fab4bda6b917d1391200c.png)

用这句话来十分巧妙的判断了当前是否有使用new操作符。

然后通过下面这几个方法来按类型为Vue继续添加一些重要的继承属性和方法，比如生命周期。

![](/images/posts/2019-09-04-VueSource-vueYuanMaFenXi(1)/87c628052b48cf28491bb2e6b94ee442.png)
