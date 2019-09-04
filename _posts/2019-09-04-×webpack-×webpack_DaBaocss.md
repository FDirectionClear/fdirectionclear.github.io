---
layout: post
title: 打包css
date: 2019-09-04
tag: webpack
---

### 一、css-loader和style-loader打包css文件

webpack是不默认如果想要打包.css文件，我们就需要借助css-loader

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/a2f5d0b325642ac935ac77fc875ca38a.png)

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/0e60488d5deaf455a497f61f0b6a071d.png)

就需css-loader(先)和style-loader（后），我们先安装它：

Npm install style-loader css-loader --save-dev

配置如下：

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/2857f08e54a4474282caacbfd62bcf29.png)

其中css-loader是用来解析.css文件用的，在解析完成之后，他会把解析过的内容交给style-loader，style-loader就会在index.html的head标签内创建一个style标签，并把解析好的css代码插入到index.html的style标签内。**因此从逻辑的角度来看，css-loader应该是率先执行的，style-loader则是处理css-loader处理好的结果。**

因此我们必须注意css-loader和style-loader的引入顺序。

实验效果：

1.  **css文件编写**

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/cef60e1ba5a657840de8276a79d17fba.png)

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/ee9bad28b94fd3515729ce50492887a0.png)

**2、index.html展现效果**

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/aabe9def37ed2c3d296098e3bd963de3.png)

**3、打开控制台查看head中的style标签**

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/16b8b7b35a33446a4e911a6af20ad2b7.png)

可以看到背景图片以及他的样式都被应用到了index.html中

*TIP:*

*解析.css文件需要style-loader和css-loader的配合。并且要有一定的先后顺序。不要忘记loader的安装时从下往上，从右向左的。应该先使用css-loader来解析import的.css文件，并且能分析需要打包的css文件和其他css文件的引用关系，最后将各个css智能的合并在一起，之后在使用style-loader*

### 二、打包Sass文件

如果我们要打包sass，只需要安装sass-loader以及一定要和他一起安装的node-sass，然后进行配置即可。

**Sass-loader可以理解为是把.scss文件中的代码“翻译”为css代码，然后把css代码传给css-loader，所以需要注意的是sass-loader应该放到最前面！**

npm install sass-loader node-sass --save-dev

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/020fe3f3403052aedfc7943ef511afc0.png)

### 三、postcss-loader && autoprefixer

如果使用css属性，难免会使用浏览器厂商前缀，比如-o- -moz- -webkit- -ms-。

当然可以再编写代码的时候自动添加，但是这样会很乏力。

尽管在.vue文件中编写的样式文件已经智能的加上了当前浏览器所需要的前缀。

但是我们自己在.vue外部文件中编写的css代码就不能享受这个福利了。因此我们需要在编译css时候就自动加上厂商前缀。

**完成这项任务的是postcss-loader和他的好伙伴autoprefixer插件。**

可以使用postcss-loader并结合autoprefixer插件来对编写的css3属性智能的添加浏览器厂商前缀，所以在写css3属性的时候可以不用考虑浏览器厂商前缀的问题了。

他们的使用方法：

**1、首先安装postcss-loader和autoprefixer插件**

npm i postcss-loader autoprefixer -D

**2、配置postcss.config.js**

和babel类似，postcss可以独立出一个配置文件，在webpack.config.js同级目录下

1.  **创建postcss.config.js。**

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/8f926a8f23e6572671fbdc4262b832af.png)

**（2）配置postcss.config.js:**

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/06673715761be0ba63d964aa83a97884.png)

### 3、在webpack中配置

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/96f506ba14cf4fdd0963fcd81b514eb4.png)

**4、编写需要前缀的sass代码**

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/9b3128503542b9d4eef852fd69c4f879.png)

浏览器中生成的效果为：

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/5948caf1092361d1a0ba75bdbe88d474.png)

这里是谷歌浏览器，明显自动添加了-webkit-的厂商前缀。

### 四、项目中遇到的坑

当我们在Sass中使用\@import语句引入了其他的Sass文件的时候，可能就会直接跳过最下方的两个loader,也就是postcss-loader和sass-loader，直接去使用css-loader。至于为什么，现在还想不通，老师也没有详细指出。之后需要额外进行探索。

解决方式正是为css-loader配置一个选项importLoaders:2。意思是在执行css-loader之前还需要执行两个loader。这是就能保证执行postcss-loader和sass-loader了。

根据网上的问题反馈发现，有些时候可能会遇到了及时按照下面方式也无法自动添加厂商前缀的问题，**只是由于postcss属于后先加载器，只需要让postcss-loader与sass-loader调换一下位置即可。**对于这一点，我目前也只是一知半解，至于具体原因需要日后再深入调查。

![](/images/posts/2019-09-04-×webpack-×webpack_DaBaocss/b53a0b609440aa7db4801d37cd914540.png)
