---
layout: post
title:  "rails中yield的用法"
date:   2016-07-20 20:30:00
categories: rails
comments: true
---

yield是与ruby中的block紧密结合的，block的语法就是{}(单行)或者do...end(多行)

一个block总是被一个具有相同名称的函数调用，这就是说如果你有个名为test的block，你就可以使用名为test的函数来调用这个block.

yield语法的一个例子:

{% highlight ruby %}
def test
   puts "You are in the method"
   yield
   puts "You are again back to the method"
   yield
end
test {puts "You are in the block"}
{% endhighlight %}

打印结果：

{% highlight ruby %}
You are in the method
You are in the block
You are again back to the method
You are in the block
{% endhighlight %}

这样当执行test方法时，yield会去查找并调用名称为test的block

当然yield后面也可以有参数

{% highlight ruby %}
def test
   yield(2)
end
test {|i| puts "This index is #{i}"}
{% endhighlight %}

打印结果：

{% highlight ruby %}
This index is 2
{% endhighlight %}

也可以传多个参数如:

{% highlight ruby %}
yield(a,b)
test{|a,b|  puts "#{x}, #{y}"}
{% endhighlight %}

另一种写法：

{% highlight ruby %}
def test(&block)
   block.call
end
test { puts "Hello World!"}
{% endhighlight %}

﻿调用block的两种方式:

{% highlight ruby %}
1.单行block: test { .... }
2.多行block:
test do
   ....
end
{% endhighlight %}