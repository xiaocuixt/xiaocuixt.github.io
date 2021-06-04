---
layout: post
title:  "Ruby中的OpenStruct和Struct"
date:   2017-05-21 16:30:00
categories: ruby
comments: true
---
<h3> OpenStruct </h3>
<p>如官方文档介绍, OpenStruct是一个类似Hash的数据结构,它允许我们使用对象自带的值定义任意的属性,
<p>这是通过使用Ruby的元编程来定义类本身的方法来实现的.
<p>说白了就是通过一个数据结构来模拟class,有些数据有自己的属性，但是使用class会比较复杂，使用hash即可满足，但是hash存取值又不太方便，这是OpenStruct就排上用场了。

{% highlight ruby %}
require 'ostruct'
post = OpenStruct.new(title: "My first post", content: "My first post content")
# 三种读取方式返回相同的值
post.title    #=>  "My first post"
post[:title]  #=> "My first post"
post['title']  #=> "My first post"
{% endhighlight %}

<p> OpenStruct类也包含很多类方法:
<p> to_h可以方便的转化为hash, 如:
<p> post.to_h  #=>  {:title=>"My first post", :content=>"My first post content"}
delete_field可以删除某个元素:

{% highlight ruby %}
post.delete_field(:content)  #=>  {:title=>"My first post"}
{% endhighlight %}

<p> 虽然在Ruby中OpenStruct封装的很好用，但是有性能性能，具体参考：https://stackoverflow.com/questions/1177594/when-should-i-use-struct-vs-openstruct#answer-4459132



<h3>Struct</h3>

Struct本质上是一个简单的类,它允许你在不用定义class的情况下封装属性和accessor方法。

创建一个Struct:

{% highlight ruby %}
Post = Struct.new(:title, :content)
post = Post.new('my title', 'my content')
# 三种读取方式返回相同的值:
post.title    #=>  "My first post"
post[:title]  #=> "My first post"
post['title']  #=> "My first post"
{% endhighlight %}

<p> Struct的方法：

{% highlight ruby %}
post.values  #=> ["my title", "my content"]
post.to_h  #=> {:title=>"my title", :content=>"my content"}
{% endhighlight %}

<p> Struct比OpenStruct更优的地方是在Struct内部可以定义方法:

{% highlight ruby %}
Post = Struct.new(:title, :content) do
   def display_title
      title.to_s + " created at today"
   end
end
Post.new("my title", "my content").display_title  #=> "my title created at today"
{% endhighlight %}


<p> 什么时候使用Struct？

<p> Struct像是一个简单的封装了属性和方法的类.

<p> 那么Struct的适用场景是什么呢

<p> 使用Struct最大的好处是它可以比hash或者array更好的表明数据的意义.

<p> 比如，假设你有下面记录坐标数据的数组:

{% highlight ruby %}
locations = [[40.748817, -73.985428], [40.702565, 73.992537]]
{% endhighlight %}

<p> 这个数组看起来并不能很清楚的展示其中的值是代表的坐标相关的数据。

<p> 现在我们使用Struct来定义这些值:

{% highlight ruby %}
Location = Struct.new(:longitude, :latitude)
location = Location.new(40.748817, -73.985428)
location.longitude   #=> 40.748817
{% endhighlight %}

<p> 当要处理的是相关性比较强的数据时,你应该使用用Struct.比如上面的location的属性关联密切， 所以比较适合封装成一个对象.

<p> 使用Struct的类比Hash还有一个好处是Struct值require指定的属性，而Hash可以接受任何值.

<p> 比如，假设我们正在处理一些关于book的结构化的数据，如果我们使用Hash:

book = {}
book[:tile] = 'Jane Eyre'
<p> 这个Hash不管我们拼写的title是否正确，都会加到Hash中.



<p> 而使用Struct不会出现这个问题.

<p> 同样取坐标的例子:

{% highlight ruby %}
Location = Struct.new(:longitude, :latitude)
location = Location.new(40.748817, -73.985428)
location.longitud   #=> undefined method `longitud' for struct
{% endhighlight %}
﻿


<p> 参考:<a href="http://culttt.com/2015/04/15/working-with-structs-in-ruby/">http://culttt.com/2015/04/15/working-with-structs-in-ruby/</a>
