---
layout: post
title: Sass中‘&’含义自我理解
date: 2018-09-06 
tag: Sass/Less/CSS
---
## 如果在刚开始接触到Sass中的“&”符号的话，就看到以下代码：  
```
nav {
    a {
        color: red; 
    }
    header & {
        color: green; 
    }
}
```
会生成以下内容的CSS目标文件：
```
nav a {
  color:red;
}
header nav a {
  color:green;
}
```
可能会有点发蒙，虽然硬记不是很困难，但是总觉的这样有点反人类的思维，很难受。所以思考了很久，总结出易于理解的说法：
```
nav {
    a {
        color: red;  /*先假设没有下面的代码，如果说在这个文档的其他部分还有很多nav a这样的结构的话就会让它们的文字变成红色，header nav
                       a 的颜色也同样会是红色的 */
    }
    header & {
        color: green; /*加上这段代码就会很容易的强调出header nav a下的颜色，让他变成绿色，而不和其他的地方一样为红色*/
    }
}
```
这样的话就好理解多了。  
这也可以看出，**&起到了取父元素的功效**。
