---
layout: post
title: css换行
date: 2019-09-04
tag: Sass/Less/CSS
---

参考博文：<https://www.cnblogs.com/zhishaofei/p/4169622.html>

先说明一点：

如果一个div的宽高固定。如果是CJK（中日韩亚洲字体）在碰到容器边界的时候就会自动换行。对于英文，在不特殊制定css属性的情况下，一个非常长的以至于超出边界的单词是不会自动折断的（连续的字母或者数字）。

如果不是连续的，也就是是词组或者语句，那么默认规则是，每写入一个单词（连续的字母或者数字），就会看这一行后面的空格是否够下一个单词的长度使用，如果够了就继续在这行写入单词。如果不够就换到下一行。

但是如果后面的单词足够长，足以自己就超过了容器的宽度，默认情况下就没有办法了，只能让他自成一行，然后溢出容器。

如果想改变上面的规则，就不得不使用

Word-wrap:　normal \|\| break-word;

Word-break: normal \|\| keep-all \|\| break-all;

White-space: normal \|\| pre \|\| pre-wrap \|\| pre-line

\<style\>

\#xsa{

height:150px;

width:150px;

border:2px hotpink solid;

/\* word-break:-all; \*/

/\* word-wrap:break-word; \*/

/\* white-space:pre; \*/

}

\<div id = "xsa"\>

adasd wxuhwjdaxwu ehajhuwdaxuw

hdiasuywjcxuahwuasdawdawd

\</div\>

![](/images/posts/2019-09-04-SassLessCss-csshuanhang/6163c019af0544f40e91d38ce972cfec.png)

Word-wrap: break-word;

使用这一特性的时候，对字符串的处理规则只与默认处理方式有一点不同，就是当一行后面的空格无法装下下一个单词的时候，就让下一个单词放在下一行，只不过，如果这个单词本身就超过了宽度，就会让这个单词在容器边界折断。而默认的情况就会让他溢出，如上图所示。

使用了word-wrap:break-word;

![](/images/posts/2019-09-04-SassLessCss-csshuanhang/eabc88f41a2582704900c73bfc828cc5.png)

而word-break:break-all；则是只考虑边界，不考虑单词是否能放在一行里，能折行的就折行。

使用了word-break:break-all;

![](/images/posts/2019-09-04-SassLessCss-csshuanhang/bbd11efce97b304dcee896f3466d05c1.png)

如果使用了white-space，比如说pre。那么值就会产生类似于pre标签的效果。

保留了换行符与空格。
