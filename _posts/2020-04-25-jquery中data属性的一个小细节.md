---
layout: post
title:  "jquery中data属性的一个小细节"
date:   2020-04-25 18:30:00
categories: rails
comments: true
---


很多人都用过dom中的data属性，而jquery也提供了对data属性的读取和设置方法。
一般常用的是:
```
<a href="" data-name="hello name" class="link"> 
```
Get写法:  
```
 $(".link").data("name")
```
Set写法: 
```
 $(".link").data("name", "hello name2")
 或者
 $(".link").attr("data-name", "hello name2")
```
那这两个Set写法有什么区别呢?

data方法不会改变该元素的html5的"data-*"属性，jQuery在内部缓存了一个key-value形式的值，所以当你调用$(".link").data("name")时，你只是从jQuery内部读取了这个值。
```
$(".link").data("data-name", "hello name2")  #=>DOM不变化
$(".link").data("name")  #=》 "hello name2"
$(".link").attr("name")  #=》 undefined
```

而attr()方法不仅内部缓存该值，还会改变html5的"data-*"属性。所以执行结果如下:
```
$(".link").attr("data-name", "hello name2")  #=>DOM变化: <a href="" data-name="hello name2" class="link">
$(".link").attr("data-name")  #=》 "hello name2"
$(".link").data("data-name")  #=》 undefined
```
所以可以得出结论，attr()和data()在set和get值是是独立的，而如果你需要更改html5的data-*值的话，需要使用attr才能满足。



