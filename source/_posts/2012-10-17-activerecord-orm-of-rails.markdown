---
layout: post
title: "ActiveRecord--ORM of Rails(1)"
date: 2012-10-17 23:07
comments: true
categories: ruby, rails
---

最近在看ihower的实战圣经，白天不能上网的话就看《Agile Web Development with rails》的后半部分，又看了railscasts开始几集，正好都讲到了ActiveRecord,也就是rails里MVC的model部分，就索性做个总结。

##ORM
ActiveRecord是rails定义的一个类，是rails的ORM，很强大。

ORM(Object-relational mappiing)，用我自己的话来说，就是建立在数据库与web程序之间的一种抽象机制，把关系行数据库的表和程序中的对象建立关系，简化了对于数据库的操作，起到了极大的简化作用。而这种抽象与简化对于那些SQL用的不是很熟练的人，对于开发web应用有很好的方便，可以不过多关注底层的sql，而专注于web应用本身。而有些人可能会比较反对这种机制，因为效率或是其他原因。

而Joel Spolsky有个抽象渗漏法则，就是说
> 所有重大的抽象机制在某种程度上说都是有漏洞的。比如C语言简化了组合语言的繁杂，ruby简化了C，TCP简化了IP，而他们又都有一些缺陷，而他们的实现里又都会混入一些后者，来达到效率等目的。

但是，取舍在于这种抽象能带来更大的好处，比如开发时间的缩短或是怎样。比如，学习ruby、rails这些时间，确实感觉到了它们的快速和方便，虽然听说效率可能不及C或java，但是他们带来的时间的节省以及带给程序员的快乐是很大的，我觉得这些原因就足够了。

<!-- More -->

##Dynamic Find
查询是数据库操作很重要的一部分，ActiveRecord有first、last、all、find、where，以及limit、order、offset、select、group等方法。

这里就主要说一些其中的Find，因为有ruby里动态方法，所以显得很神奇和强大。

find单独用的话，可以直接传入id，如`c = Order.find(1)`来得到表的一行数据，然后经过rails处理，变为ruby的某个类型的对象。railscasts里说还可以传入参数，进行查询，不过查了其他资料，没见到用的，可能是rails3取消了，统一用动态方法了？

还有`find\_by\_sql("...")`，可以把sql语句传进去进行查询。

好了，说动态方法，其实是把要查询的条件直接作为方法名来使用。这里举几个例子：
```
User.find_by_first_name('Jack')    #根据first_name这个属性来找到first_name为Jack的User，而且这里默认是first（第一个）
User.find_all_by_first_name('Jack')   #这里则是找到所有叫Jack的人
User.find_last_by_first_name('Jack')  #找到最后一个满足条件的
User.find_by_first_name_and_locked("Jack", true)  #这里则传入了两个参数
```
这里可以把任何属性名放到by后边（多个词的话就要用'_'隔开），有点magic的感觉，其实都是因为ruby的Metaprogramming（元编程）的原因。先不说，因为还要多差点资料 ：）。

##Scope
这其实是很酷的东西，可以让查询变得很灵活，代码看起来也很简洁，之前看angle_nest的代码，看到model中的scope就惊艳了，这么好的东西以前居然不知道的。
它最帅的，就是可以把几个单独的小一点的查询连起来，以及指尖的相互组合查询。如(来自ihower)：
```
class Event < ActiveRecord::Base
    scope :public, where( :is_public => true )
    scope :recent_three_days, where(["created_at > ? ", Time.now - 3.days ])
end

      Event.create( :name => "public event", :is_public => true )
      Event.create( :name => "private event", :is_public => false )
      Event.create( :name => "private event", :is_public => true )

      Event.public
      Event.public.recent_three_days
```
还可以传入参数：
```
class Event < ActiveRecord::Base
    scope :recent, lambda{ |date| where(["created_at > ? ", date ]) } 
    # 或 scope :recent, Proc.new{ |t| where(["created_at > ? ", t ]) }
end

    Event.recent( Time.now - 7.days )
```
这里是可以查询到最近多久的Event。
不过这里，ihower和guides里都推荐换成这种形式，可读性比较强：
```
class Event < ActiveRecord::Base
    def self.recent(t=Time.now)
        where(["created_at > ? ", t ])
    end
end

Event.recent( Time.now - 7.days )
```
这种也可以和其他Scope连在一起使用。

