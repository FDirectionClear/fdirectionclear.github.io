---
layout: post
title: Javascript稳妥构造函数模式
date: 2018-09-07
tag: Javascript
---
    JSONP是一种跨域的Ajax请求，之前学习的时候遇到过也用过，今天又遇见了，就打算把他也加进Blog中。  
# 什么是Jsonp？  
    JSONP(JSON with Padding)是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题。由于同源策略，一般来说位于 server1.example.com 
的服务器是无法与不是 server1.example.com的服务器沟通，如果在用GET或者用POST请求一个不在自己服务器上的服务器，一定会导致请求失败，
这是因为Http协议中不允许这样操作，但这是很容易理解的，毕竟服务器互相交杂，访问不受限制必定会导致解决不完的安全隐患。而 HTML 的<script> 元素
是一个例外。利用 <script> 元素的这个开放策略，网页可以得到从其他来源动态产生的 JSON 资料，而这种使用模式就是所谓的 JSONP。但是需要注意的是，
用 JSONP 抓到的资料并不是 JSON，而是任意的JavaScript（是由目标服务器中的Js回调函数返回的值，这个返回值可以是JSON形式的数据），用 JavaScript
直译器执行而不是用 JSON 解析器解析。
    JSONP的请求方式不是GET和POST，而是JSONP。在Vuejs中，JSONP被抽象成了一种请求的方式，就如同GET与POST一样。  
    ```
    
    ```
