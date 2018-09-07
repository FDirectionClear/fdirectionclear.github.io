---
layout: post
title: Javascript稳妥构造函数模式
date: 2018-09-07
tag: Javascript
---
# 稳妥构造函数模式
  &nbsp;&nbsp;&nbsp;&nbsp;今天我发现了一个非常妙的对象构造模式——***稳妥构造函数模式（durable objects）***。这个概念。所谓稳妥对象，说白了就是没有公共属性，而且其方法不借助  this对象,
  稳妥多项最适合在一些安全环境中（这些环境中禁止使用this和new），或者在防止数据被其他应用程序改动时使用。
  ```
  function Person(name.job,age){
    //创建要返回的对象
    var o = new Object();
    //可以在这里定义私有变量和函数
    //添加方法
    o.sayName = function(
      console.log(name);
    );
    //返回对象
    reyurn o;
  }
  ```
