---
layout: post
title:  "namespace in javascript"
date:   2019-01-08 20:30:00
categories: javascript
comments: true
---

<p>js默认是没有namespace的，所以我们在js中定义的任何东西(function, method, object, variable)都是全局性的，甚至在编码中会不断的增加变量等污染全局作用域。幸运的是，我们可以通过Object和IIFE(立即调用的函数表达式)来模拟出我们自己的namespace。这有助于我们更好的管理代码，而不用创建一些无用的全局变量和全局函数。</p>

<p>下面的例子我们分别创建两个同名的方法:</p>

```javascript
function func1(){
    console.log("I am first func")
}
function func1(){
    console.log("I am second func")
}
func1();  //打印出 I am second fun1
```
<p>上面我们模拟在不同的地方定义两个同名的func1, 结果只运行了最新定义的那个，产生了冲突。现在我们通过将两个函数分别放到不同的namespace下处理下命名冲突，使两个方法逻辑隔离开来。</p>
<p>有几种方法可供选择: </p>

<h5>第一种: 使用Object模型来模拟</h5>
<p>我们把变量和函数包在一个充当namespace的Object里面，然后通过点号访问被包裹的变量和函数.</p>

```javascript
var MyNamespace1 = MyNamespace1 || {};
var MyNamespace2 = MyNamespace2 || {};

MyNamespace1.func1 = function(){
  console.log("I am first func")
}
MyNamespace2.func1 = function(){
  console.log("I am second func")
}

MyNamespace1.func1();  // I am first fun1
MyNamespace2.func1();  // I am second fun1
```

<h5>第二种:使用IIFE(Immediately invoked function expression) 立即调用的函数表达式</h5>

<p>IIFE是包含在一对括号内的匿名函数，它能够立即执行，这对括号为它们内部的所有代码创建了一个局部的scope，并使该匿名函数成为函数表达式。适用于定义一些不会在其它地方调用的函数</p>
<p>最外面的一对括号将其中的所有内容都转换为表达式，因为括号不能包含JavaScript语句。函数定义后的另一对括号立即调用该函数。我们来看一个例子。</p>

```javascript
(function(){
    function func1(){
      console.log("I am first func1")
    }func1();
}());

(function() {
   function func1(){
     console.log("I am second fun1");
   } func1();
 }());

 (function(){
    var x = 123;
    console.log(x);
 })(); // 直接打印出123,且只打印一次
```

<h5>第三种：使用block，let或const声明</h5>
<p>在es5中我们可以使用上面的IIFE模式。如果需要严格的限制变量的范围，在es6中可以使用block中let(或const)声明</p>

```javascript
  {
    let temp = function fun1(){
      console.log("I am first fun1");
    }
    temp(); // "I am first fun1"
  } // 实时调用

  fun1();  //报错,let声明的函数作用域为block内部，在block外部无法调用到

  {
    let temp= function fun1(){
      console.log("I am second fun1");
    }
    temp(); //"I am second fun1"
  } // 实时调用

  fun1();  //报错,let声明的函数作用域为block内部，在block外部无法调用到
```

<p>Ref:</p>
<a href="https://dev.to/muhammadridwan/namespace-in-js-5dbj">https://dev.to/muhammadridwan/namespace-in-js-5dbj</a>

<a href="https://github.com/stevekwan/best-practices/blob/master/javascript/best-practices.md">https://github.com/stevekwan/best-practices/blob/master/javascript/best-practices.md</a>