---
layout: post
title: "ActiveRecord-ORM of Rails(2)"
date: 2012-10-22 01:17
comments: true
categories: rails, ruby
---
这两天重装系统，没做什么事情，装系统现在已经很快了，只是还在做好多的后续工作，回头再写篇总结好了。先写之前剩下的一篇。这次是聊点ActiveRecord的callback。

##Monitor

ActiveRecord控制model的生命周期，包括他们的CRUD（《Agile..》上很风趣地说是watches sadly as they are destroyed. haha）。

而在这过程中的许多时刻我们都可以通过callbacks来进行一些操作.

ActiveRecord定义了十六个回调函数，十四个是`before/after_func`的形式，分别在一些函数之前或之后被调用，如`destory, validation`,两个特别的是after\_find以及after\_initialize，这两个没有对应的before。

他们的流程如下：
{% img /images/callbackactiverecord.png %}

<!-- more -->

做个简单的解释，`save()`会产生两种行为，一种是新产生，一种是更新。而这两种行为可以通过`before_validation`或`after_validation`等的方法加上参数`on: create`或`on: update`来决定是哪种。

而之前提到的`after_find`是在任何find操作后会调用，`after_initialize`则会在一个Active Record的对象产生后调用。我的理解是，比如`Model.new()`这个只是产生了一个ActiveRecord对象，并没有save，所以调用`after_initialize`。

怎么用呢？ 我们需要写一个handler和一个与它关联的相应的回调函数。

比如：
```
class Order < ActiveRecord::Base
  before_validation :normalize_credit_card_number
  after_create do |order|
    logger.info "Order #{order.id} created"
  end
  protected
  def normalize_credit_card_number
    self.cc_number.gsub!(/[-\s]/, '')
  end
end
```

在`before_validation normalize_credit_card_number`中声明了一个handler并把它关联到`before_validation`这个事件中。然后再定义一个函数就行，这个函数最好是private或protected的。
