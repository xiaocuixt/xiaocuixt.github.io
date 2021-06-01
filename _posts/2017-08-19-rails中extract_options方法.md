---
layout: post
title:  "rails中extract_options方法"
date:   2017-08-19 22:30:00
categories: ruby
comments: true
---

<p>
官方介绍:
Extracts options from a set of arguments. Removes and returns the last element in the array if it's a hash, otherwise returns a blank hash.
</p>
<p>
extract_options!不是一个标准的ruby方法，而是一个rails为Array类提供的拓展方法，需要引用ActiveSupport才能使用该方法，以!结尾，该方法是一个"Bang"方法，会改变调用对象的值.
</p>
<p>需要注意的是，如果参数中没有hash，该方法返回一个空的hash</p>

<p>数组直接调用</p>

```ruby
a = [1, 2, a: 3]
a.extract_options!  #=> {:a=>3}
a #=> [1, 2] ## 数组a的值被改变
```

<p>参数形式</p>

```ruby
def options(*args)
  args.extract_options!
end

options(1,2)  #=> {}
options(1,2,a: :b) #=> {:a => :b}
options(1,2,a: :b, c: :d) #=> {:a=>:b, :c=>:d}
```

<p>我们经常在rails中见到类似下面的方法调用的写法:</p>

```ruby
custom_method :arg1

custom_method :arg1, arg2, arg3

custom_method :arg1, shown: true, age: 25
```

<p>
  我们可以看到上面的custom_method可以传入任何数量的参数，后面跟着一个或者多个options.
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

```ruby
before_action :my_method, if: :excute?, only: %w(new)

validates_presence_of :field, :allow_blank: true
```
<p>源码也比较简单：
<a href="https://github.com/rails/rails/blob/08754f12e65a9ec79633a605e986d0f1ffa4b251/activesupport/lib/active_support/core_ext/array/extract_options.rb">查看源码</a>
</p>

