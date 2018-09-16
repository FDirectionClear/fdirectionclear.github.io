---
layout: post
title: 关于Vue使用全局组件填坑
date: 2018-09-06 
tag: Vue.js
---
&nbsp;&nbsp;最近使用了Vue2.0中的全局组件，遇到了一些问题，但是经过思考，这只是犯了一个低级的错误，已经自我思考并反省出问题的所在，现总结如下：  
# Vuejs2.0全局组件声明  
## 方式一：先定义组件内容，再进行组件注册  
&nbsp;&nbsp;<font color="#dd0000">**使用Vue.expend({template:"/** some DOM Elememt **/"})定义组件内容，并把返回的内容传给一个变量：**</font>  
```
  var login = Vue.extend({  //定义组件内容
    template:"<h1>登陆组件</h1>"   
  });
  
```  
&nbsp;&nbsp;**使用Vue.component("/** tag-name **/",targetContent)注册组件：**  
```
  Vue.component("login",login);  //注册组件
  
```
&nbsp;&nbsp;**最后在DOM中使用组件元素即可：**
```
   <div id = "app1"> 
        <register></register>  //使用组件元素
        <button @click = "toggle">toggle</button>
        <transition name = "fxm">
            <div class = "move-box" v-show = "show"></div> 
        </transition>
    </div>
    <div id = "app2">
        <register></register>  //使用组件元素
    </div>
```
## 方式二：定义组件与注册组件一气呵成
&nbsp;&nbsp;**将组件的内容作为第二个参数直接代入Vue.component("tag-name",{template: /** some DOM Element **/})**  
```
  var login = Vue.component("register",{
    template:"<div><h1>注册组件</h1></div>"
  });
  
```  
&nbsp;&nbsp;**之后使用组件的操作与方式一相同**
## 方式三：使用template元素作为模板直接在DOM中定义组件内容（最佳推荐）
&nbsp;&nbsp;**声明与定义组件的函数相同，只不过是组件内容的template属性的值变为一个指向DOM模板的指针**
```

  Vue.component("sign-out-1",{
      template:"#model-1"  //id选择器指向DOM中的模板元素
  });

  Vue.component("sign-out-2",{
      template:"#model-2"  //id选择器指向DOM中的模板元素
  });
  
```  
```
  <template id = "model-1"> <!--模板内容不会显示在视图中-->
      <span><a href="#">登陆</a> | <a href="#">注册</a></span> <!--注意需要一个root元素！root元素！-->
  </template>

  <script type = "x-template" id = "model-2"> <!--模板内容不会显示在视图中-->
      <span><a href="#">登陆2</a> | <a href="#">注册2</a></span> <!--同样需要一个root元素，但是在script标签中代码不会高亮-->
  </script>
  
  <sign-out-1></sign-out-1> <!--使用组件-->
  <sign-out-2></sign-out-2> <!--使用组件-->

```
<font color=#00ffff size=72>颜色设置</font><br/>

# 需要注意的坑
