---
layout: post
title: Sass中混合宏VS继承VS占位符
date: 2018-10-04
tag: Sass
---
  **<font color="red">初学者都常常纠结于这个问题“什么时候用混合宏，什么时候用继承，什么时候使用占位符？”其实他们各有各的优点与缺点，先来看看他们使用效果：</font>** 
  
  a) Sass 中的混合宏使用
  ```
  //SCSS中混合宏使用
@mixin mt($var){
  margin-top: $var;  
}

.block {
  @include mt(5px);

  span {
    display:block;
    @include mt(5px);
  }
}

.header {
  color: orange;
  @include mt(5px);

  span{
    display:block;
    @include mt(5px);
  }
}
  ```  
  
  编译出来的 CSS 见右侧结果窗口。
  ```
  .block {
  margin-top: 5px;
}
.block span {
  display: block;
  margin-top: 5px;
}

.header {
  color: orange;
  margin-top: 5px;
}
.header span {
  display: block;
  margin-top: 5px;
}
  ```  
  可以看到margin-top的代码发生了代码冗余，如果
  ```
  .block,block span,header,header span{
    margin-top:5px;
  }
  ```
  就可以解决问题，但是很遗憾Sass的混合宏并不能只能的将冗余的属性合并在一起。  
  
  个人建议：如果你的代码块中涉及到变量，建议使用混合宏来创建相同的代码块。因为混合宏是可以传递参数的，传参也是混合宏不同于另外两个最大特点。
  
  b) Sass 中继承  
  
  同样的，将上面代码中的混合宏，使用类名来表示，然后通过继承来调用：
  
  ```
  //SCSS 继承的运用
  .mt{
    margin-top: 5px;  
  }
  
  .block {
    @extend .mt;
  
    span {
      display:block;
      @extend .mt;
    }
  }
  
  .header {
    color: orange;
    @extend .mt;
  
    span{
      display:block;
      @extend .mt;
    }
  }
  ```
  就会编译出如下的CSS代码：
  ```
  .mt, .block, .block span, .header, .header span {
  margin-top: 5px;
}

.block span {
  display: block;
}

.header {
  color: orange;
}
.header span {
  display: block;
}
  ```
 因此，可以看出继承有着比混合宏更智能的一点——可以合并冗余的代码。将继承了同样基元素的所有元素都列在一起共享一个属性。而他们这些元素独有的再而外重新写。

 但是缺点是在CSS文件中生成了额外的基元素选择器。如果这个基元素确实存在于HTML中那还好，否则就要写一堆基元素选择器。最终可以得出结论，基元素确实在HTML中
 
 存在，还不想多生成重复的代码，就是用继承。
 
 c) 占位符
 最后来看占位符，将上面代码中的基类 .mt 换成 Sass 的占位符格式：
 ```
   //SCSS中占位符的使用
  %mt{
    margin-top: 5px;  
  }
  
  .block {
    @extend %mt;
  
    span {
      display:block;
      @extend %mt;
    }
  }
  
  .header {
    color: orange;
    @extend %mt;
  
    span{
      display:block;
      @extend %mt;
    }
  }
 ```
 编译生成一下CSS代码：
 ```
 .block, .block span, .header, .header span {
  margin-top: 5px;
}

.block span {
  display: block;
}

.header {
  color: orange;
}
.header span {
  display: block;
}
 ```
 可以看出，上面的结果与继承很相似，拥有者只能合并多余代码的功能，并且不会在CSS中出现基元素 %mt！
 
 如果没有调用占位符，那么Sass文件中的占位符就会和他的属性默默沉睡在Sass中，不会出现在CSS文件中。
 
  
