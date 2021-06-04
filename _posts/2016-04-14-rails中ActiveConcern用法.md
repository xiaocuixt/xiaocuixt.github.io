---
layout: post
title:  "rails中ActiveConcern的用法"
date:   2016-04-14 18:30:00
categories: rails
comments: true
---
首先了解下mixin的概念:

mixin混入类,一个mixin可以看作是一组代码,可以添加到一个或者多个类中，在不使用继承的情况下,为这些类添加额外的功能.

在ruby中一个mixin是一个封装好的module,其他class可以include或者extend该module.事实上,

一个class可以拥有很多mixin.

model和class中的方法:

class中有实例方法和类方法两种。

module中也有两种方法称之为module方法和没有module名字的方法,其中module方法可以直接被module调用,

没有module名字的方法不能直接调用，需要mixin到一个类中。

1.  先看下ruby中include, extend方法：

include方法，例子:

{% highlight ruby %}
module Person
  def name
      puts "my name is Person"
  end
end
class User
   include Person
end
{% endhighlight %}

执行  User.new.name  =>  "my name is Person"

这里include所做的就是将在module内定义的方法在一个class的实例变量上可用。

如果在 class user中使用extend Person,则可以使用User.name,即为User添加了一个类方法。

extend用法例子:

{% highlight ruby %}
module User
  extend self
  def say
    'hello'
  end
end
{% endhighlight %}

这样定义的方法可以为User所调用， User.say  => "hello"

2. ActiveSupport::Concern  是为了管理modules之间的依赖关系，而在class中include时，不需要关心module之间的关系。

一个普通的module写法：

{% highlight ruby %}
module M
  def self.included(base)
    base.extend ClassMethods
    base.class_eval do
      scope :disabled, -> { where(disabled: true) }
    end
  end
  module ClassMethods
    ...
  end
end
{% endhighlight %}

使用ActiveSupport::Concern重构上面的module:

{% highlight ruby %}
require 'active_support/concern'
module M
  extend ActiveSupport::Concern
  included do
    scope :disabled, -> { where(disabled: true) }
  end
  class_methods do
    ...
  end
end
{% endhighlight %}

3. 下面是一个项目中实用的例子:

Model  Concern:   虽然我们主张“fatter model,thinner controller”,但是model中的代码也不宜过多,

好的解决方法就是把相关的额逻辑代码放到对应的ActiveSupport::Concern里面去,提高代码的高内聚,低耦合。

{% highlight ruby %}
class Post < ActiveRecord::Base
  # scopes
  scope :active, -> {where(is_active: true)}
  # instances methods
  def active?
    is_active
  end
end
class Advertisement < ActiveRecord::Base
  # scopes
  scope :active, -> {where(is_active: true)}
  # instances methods
  def active?
    is_active
  end
end
{% endhighlight %}

两个模型具有相同逻辑的代码可以提取到ActiveConcern里面，这样多个model可以共用，遵循DRY原则.

{% highlight ruby %}
module ActAsActivable
  extend ActiveSupport::Concern
  included do |base|  #这里的base参数代表的是include该module的class
    scope :active, -> {where(is_active: true)}
  end
  # instance methods
  def active?
    is_active
  end
end
{% endhighlight %}

然后我们在model中include即可:

{% highlight ruby %}
class Post < ActiveRecord::Base
  # concerns
  include ActAsActivable
end
class Advertisement < ActiveRecord::Base
  # concerns
  include ActAsActivable
end
{% endhighlight %}

大牛总结：

concern将部分可重用的功能抽出来，然后多个model可以共用model的代码太多,将其相关的逻辑代码放到不同的concern里

规定使用ActiveSupport::Concern的代码风格,是希望形成开发规约定,就如controller和model的写法。

Controller Concern  在controller里使用Active Concern：

可以参考gem： https://github.com/josevalim/inherited_resources

项目实战，项目中统一使用deleted_at进行软删除，可以将scope方法和删除复原操作提取到ActiveSupport::Concern中:

{% highlight ruby %}
app/models/concerns/self_deleteable.rb

module SelfDeletedable  
   extend ActiveSupport::Concern 
   scope :enabled, -> {where(deleted_at: nil)} 
   included do 
     def delete  
          self.update_attributes(deleted_at: Time.now)     
      end    
      def restore        
          self.update_attributes(deleted_at: nil)    
       end
   end
end
{% endhighlight %}

在spp/models/worker.rb中引入SelfDeletedable module

{% highlight ruby %}
class Worder < ActiveRecord::Base
   include SelfDeletedable
end
在app/controllers/workers_controller.rb:
def delete
   @worker.delete
end
{% endhighlight %}

参考：https://ruby-china.org/topics/19812

这里讲了如果不同的model都有共同的is_active字段管理状态scope时的重构.