---
layout: post
title: "Use include for querying to optimize"
date: 2012-10-29 23:06
comments: true
categories: [rails, query, optimize]
---

今天写项目，然后参考ruby-china的一部分代码的时候，看到一个`include`的用法，不知道是怎么回事，因为它是用mongodb的，所以也一时找不到`ActiveRecord`中的用法。

不过刚刚在看railscasts的时候，突然就看到了这种用法:

比如说`Task`和`Project`是多对一的关系

在index的页面需要用到`Task`和它`belongs_to`的那个`Project`，这样的话，看log里，会发现，查找了`Project`了很多次。

那么就能用`@tasks = Task.find(:all, :include => :project)`来查到和project关联的task，这样的话可以把相关联的对象也先读取出来。

那么有什么结果呢？

在console的log里就看得很清楚了，载入页面之后，两次查询就够了。

我自己去写的时候，`find`方法不起作用，不知道是因为这是以前的写法还是怎样。
就看到实战圣经上有说这个的，原来是`includes`方法，比如：

```
@events = Event.includes(:user).order('start_date DESC').page(params[:page])
```

这样就能查询到最近的`event`,并把对应的`user`包含进去，再进行分页了。顿时感觉世界好美好的有木有！！ 哈哈~

