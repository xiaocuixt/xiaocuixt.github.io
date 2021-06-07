---
layout: post
title:  "Ruby中的OpenStruct和Struct"
date:   2017-05-21 16:30:00
categories: ruby
comments: true
---
<p>Struct是一个ruby内置类，是一种将多个属性绑定在一起的快捷方式，可以让你在不用声明类的情况下使用accessor方法。Struct会生成一个新的管理一组属性的值的子类，并给每个属性创建一个setter, getter方法。</p>

```ruby
Post = Struct.new(:title, :content)  # 定义类#<Class:0x007fbf235ad540>
post = Post.new('my title', 'my content')

# 多种读取方式:
post.title    #=>  "My title"
post[:title]  #=> "My title"
post['title'] #=> "My title"
post[0]       #=> "My title"

# 像hash一样获取所有的values
post.values  #=> ["my title", "my content"]

# 可以直接转化为hash，array
post.to_h  #=> {:title=>"my title", :content=>"my content"}
post.to_a  #=> ["my title", "my content"]

post.length #=> 2  # Returns the number of struct members.
post.members #=> [:title, :content]  # Returns the struct members as an array of symbols
```

<p>定义一个匿名Struct</p>

```ruby
# Create a structure with a name under Struct
Struct.new("Customer", :name, :address)
#=> Struct::Customer
Struct::Customer.new("Dave", "123 Main")
#=> #<struct Struct::Customer name="Dave", address="123 Main">
```
<p>定义一个具名Struct, 该对象在Struct中作为一个常量存在，这就要求在系统中要唯一且以大写字母开头</p>

```ruby
# Create a structure named by its constant
Customer = Struct.new(:name, :address)
#=> Customer
Customer.new("Dave", "123 Main")
#=> #<struct Customer name="Dave", address="123 Main">
```

<p>像上面的先声明attributes然后调用时赋值看起来不太直观，新版的ruby加上了keyword方式更容易阅读和编写,即keyword_init参数</p>

```ruby
Customer = Struct.new(:name, :address, keyword_init: true)  #=> Customer(keyword_init: true)
Customer.new(name: "Dave", address: "123 Main")
#=> #<struct Customer name="Dave", address="123 Main">
```

<p>通过传入一个block为Struct内部创建一个实例方法。</p>
```ruby
Customer = Struct.new(:name, :address) do
  def greeting
    "Hello #{name}!"
  end
end
Customer.new("Dave", "123 Main").greeting  #=> "Hello Dave!"
```


<p>注意参数数量</p>
不同于常规类initialize传入参数的方式，Struct传入参数的数量要小于或等于声明的参数数量，未传入的默认为nil,超过将会报ArgumentError错误

```ruby
Customer = Struct.new(:name, :address)
Customer.new("Dave", "123 Main")
#=> #<struct Customer name="Dave", address="123 Main">
Customer["Dave"]
#=> #<struct Customer name="Dave", address=nil>
Customer["Dave1", "456"]
#=> #<struct Customer name="Dave1", address="456">
Customer["Dave1", "456", "a"]
#=> ArgumentError (struct size differs)
```

<h3> 属性名拼写错误的区别 </h3>
<p> 使用Struct的类比Hash和OpenStruct还有一个好处是Struct值require指定的属性，而Hash和OpenStruct可以接受任何值.</p>
```ruby
# Hash
book = { title: "test book", cover: "xx.img" }
book[:tile] #=>nil

# Struct
Struct.new("Customer", :name, :address)
a = Struct::Customer.new("Dave", "123 Main")
a.nme #=> undefined method `nme`

# OpenStruct
post = OpenStruct.new(title: "demo title")
post.titl  #=> nil
```

<p> 这个Hash不管我们拼写的title是否正确，都会加到Hash中.而使用Struct不会出现这个问题.

<p>当你要用到继承的时候不要用Struct, Struct像是一个简单的封装了属性和方法的类.使用Struct最大的好处是它可以比hash或者array更好的表明数据的意义.</p>


<h3> OpenStruct </h3>
<p>如果需要构建一个一次性的对象，可以选择使用OpenStruct.如官方文档介绍, OpenStruct是一个类似Hash的数据结构,它允许我们使用对象自带的值定义任意的属性,这是通过使用Ruby的元编程来定义类本身的方法来实现的.说白了就是通过一个数据结构来模拟class,有些数据有自己的属性，但是使用class会比较复杂，使用hash即可满足，但是hash存取值又不太方便，这是OpenStruct就排上用场了。</p>

```ruby
require 'ostruct'
post = OpenStruct.new(title: "My first post", content: "My first post content")
# 三种读取方式返回相同的值
post.title    #=>  "My first post"
post[:title]  #=> "My first post"
post['title']  #=> "My first post"
```

<p> OpenStruct类也包含很多类方法:

```ruby
# 可以方便的转化为hash
post.to_h  #=>  {:title=>"My first post", :content=>"My first post content"
# delete_field删除某个元素:
post.delete_field(:content)  #=>  {:title=>"My first post"}
```

<p>虽然在Ruby中OpenStruct封装的很好用，但是性能比较差. 在性能要求比较高的程序中不建议使用。具体benchmark测试见:https://stackoverflow.com/questions/1177594/when-should-i-use-struct-vs-openstruct#answer-4459132</p>
﻿
<p>记录那么多，总结一下Struct & OpenStruct的区别:</p>
<p>1. Struct新建的是一个带有预设属性的类, OpenStruct创建的是一个带有属性和值的对象</p>
<p>2. 性能上Struct比OpenStruct好很多，而OpenStruct比Struct要灵活。</p>

<p>所以在我们的日常开发中，我们会经常见到Struct和OpenStruct，所以了解他们区别既方便我们组织代码，又避免一些使用上的错误。</p>

<p>参考:</p>
<p><a href="http://culttt.com/2015/04/15/working-with-structs-in-ruby/">http://culttt.com/2015/04/15/working-with-structs-in-ruby/</a></p>
<p><a href="http://palexander.posthaven.com/ruby-data-object-comparison-or-why-you-should-never-ever-use-openstruct">http://palexander.posthaven.com/ruby-data-object-comparison-or-why-you-should-never-ever-use-openstruct</a></p>

