---
layout: post
title:  "rails中extract_options方法"
date:   2017-08-19 22:30:00
categories: ruby
comments: true
---

<p>extract_options顾名思义提取options, extract_options!方法允许你轻易地从一个数组参数中提取一系列的options。</p>

<p>
extract_options!不是一个标准的ruby方法，而是一个rails提供的拓展方法，需要引用ActiveSupport才能使用该方法，以!结尾，该方法是一个"Bang"方法，会改变调用对象的值.
</p>



<p>rails中的extract_options!方法的作用是:  从一些参数中提取options，当参数是hash时移除并且返回数组的最后一个元素，否则返回一个空的hash</p>

```
def options(*args)
  args.extract_options!
end

options(1,2)  #=> {}

options(1,2,a: :b) #=> {:a => :b}
```


<p>我们经常在rails中见到类似下面的方法调用的写法:</p>

```
my_method :arg1

my_method :arg1, arg2, arg3

my_method :arg1, :shown => true, :age => 25
```

<p>
  我们可以看到上面的my_method可以传入任何数量的参数，后面跟着一个或者多个hash类型的options.
</p>


<p>
这种实现依赖于ActiveSupport提供的extract_options!方法，该方法将options从给定的参数集合中提取出来，如果参数中没有options，该方法返回一个空的hash:
</p>

<p>
看一个例子：
</p>

{% highlight ruby %}
def my_method(*args)
  options = args.extract_options!
  puts "Arguments: #{args.inspect}"
  puts "options: #{options.inspect}"
end

my_method(1,2)
  #=> Arguments: [1,2]
  #=> options: {}

my_method(1,2, :a=>:b, :c=>:d)
  #=> Arguments: [1,2]
  #=> options: { :a=>:b, :c=>:d}
{% endhighlight %}

<p>
  extract_options!方法在rails项目中大量使用，比如filter和validate相关代码
</p>

```
before_action :my_method, :if => :excute?, :only => %w(new)

validates_presence_of :field, :allow_blank => true
```


