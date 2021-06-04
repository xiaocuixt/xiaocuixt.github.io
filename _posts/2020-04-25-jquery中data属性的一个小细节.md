---
layout: post
title:  "jquery中data属性的一个小细节"
date:   2020-04-25 18:30:00
categories: javascript
comments: true
---
<p>很多人都用过html5中的data属性，而jquery也提供了对data属性的读取和设置方法。</p>
<p>示例:</p>

```javascript
<a href="" data-name="hello name" class="link">

// Get data attribute:
$(".link").data("name")
// Set data attribute:
$(".link").data("name", "hello name2")
// OR
$(".link").attr("data-name", "hello name2")
```

<p>看似一样，那这两个Set写法有什么区别呢?</p>

<p>翻看文档发现,data方法不会改变该元素的html5的"data-*"属性，jQuery在内部缓存了一个key-value形式的值，所以当你调用$(".link").data("name")时，你只是从jQuery内部读取了这个值。</p>
<p>示例:</p>

```javascript
$(".link").data("data-name", "hello name2")  // DOM不变化
$(".link").data("name")  //=> "hello name2"
$(".link").attr("name")  //=> undefined
```

<p>而attr()方法不仅内部缓存该值，还会改变html5的"data-*"属性。</p>
<p>示例:</p>

```javascript
$(".link").attr("data-name", "hello name2")
// DOM发生变化: <a href="" data-name="hello name2" class="link">

$(".link").attr("data-name")  // "hello name2"
$(".link").data("data-name")  // undefined
```

<p>得出结论，attr()和data()在set和get值是是独立的，而如果你需要更改html5的data-*值的话，需要使用attr才能实现。</p>



