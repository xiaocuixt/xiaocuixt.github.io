---
layout: post
title:  "rails中创建rake任务"
date:   2016-04-18 16:30:00
categories: rails
comments: true
---

在rails项目中,rake是一个自动化任务的构建工具，rake用来编写任务脚本，让我们在cli中可以执行，提供了良好的任务编写结构，并且方便执行相互依赖的任务

rails项目中，rake文件存放位置：lib/tasks/user.rake

1.without namespace:

{% highlight ruby %}
task :sayhi do
  puts "hi"
end
{% endhighlight %}

执行  rake sayhi  即可

2.with namespace

{% highlight ruby %}
namespace :user do
   desc "genertate users"    #描述性文字
   task :generate => :environment do   #environment表示可指定环境 RAILS_ENV=production rake user:generate
       u = User.new([{name: "tom"},{name: "jack"}])
       u.save
   end
end
{% endhighlight %}

这样直接运行rake user:generate 即可执行生成user数据的任务

3.调用外部的rake方法

{% highlight ruby %}

#lib/tasks/hello.rake

namespace :say do
  task :hello do
    puts "hello xiaocui"
  end
end

#lib/tasks/hi.rake

namespace :say do
  task :hi do
    Rake::Task['say:hello'].invoke   #调用外部的task
    puts "hi xiaocui"
  end
end
{% endhighlight %}

执行  rake say:hi   将打印出 "hello xiaocui" 和 "hi xiaocui"

4.单次执行多个方法

{% highlight ruby %}
#lib/myweekend.rake

task :morning do
  puts "get up"
end
task :afternoon do
  puts "go shopping"
end
task :evening do
  puts "have supper"
end
task :my_weekend => [:morning, :afternoon, :evening] do
   puts "This is my whole day on my weekend"
end
{% endhighlight %}

执行rake my_weekend 将依次打印每个方法的内容:

{% highlight ruby %}
get up
go shopping
have supper
This is my whole day on my weekend
{% endhighlight %}

5.rake支持默认的任务

{% highlight ruby %}
task :default => 'say:hello'
{% endhighlight %}

执行rake 即可运行该任务，很少用。

﻿适用场景:

1.调整线上的数据,执行rake脚本可以修改production数据库数据

2.创建开发用的测试数据