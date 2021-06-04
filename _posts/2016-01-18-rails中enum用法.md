---
layout: post
title:  "rails中enum用法"
date:   2016-01-18 14:41:00
categories: rails
comments: true
---
enum是rail中提供的一个枚举方法，在很多情况下还是很方便的，相关用法做个记录：

1.假设管理post表的状态status,状态是固定的，可用integer类型

在migration里：  t.integer :status, default: 0

2.在post.rb中添加 enum status: {active: 0, unactive: 1}

这样会给Post添加三个类方法， active,unactive和statuses(status的复数形式)

给post对象添加两个方法:  active?  和unactive?

3.Post.active会查找,status的值为0的所有对象
![alt tag](http://api.photo.yunpan.360.cn/intf.php?method=AllPhoto.getThumb&qid=165602315&nid=145309830661548299&size=1280_1280&devid=&rtick=1453098306&v=1.0.1&devtype=web&sign=cc9c9e6f8d4687cb7304481de50e00db)

Post.statuses会返回状态的hash
![alt tag](http://api.photo.yunpan.360.cn/intf.php?method=AllPhoto.getThumb&qid=165602315&nid=145309830561548298&size=1280_1280&devid=&rtick=1453098306&v=1.0.1&devtype=web&sign=cc9c9e6f8d4687cb7304481de50e00db)

我们在controller里查找时可以使用

Post.where("status = ?", Post.statuses[:active])  和Post.active相同

Post.where("status <> ?", Post.statuses[:active])  #<>表示不等于，在sql语句中通用

4.post对象的active和unactive方法

post = Post.find 1

post.active?   =>  true or false

post.unactive?   =>  true or false

5.返回值不同

Post.first.status  =>  "active"

Post.first  => ![alt tag](http://api.photo.yunpan.360.cn/intf.php?method=AllPhoto.getThumb&qid=165602315&nid=145309830561548297&size=1280_1280&devid=&rtick=1453098306&v=1.0.1&devtype=web&sign=cc9c9e6f8d4687cb7304481de50e00db)