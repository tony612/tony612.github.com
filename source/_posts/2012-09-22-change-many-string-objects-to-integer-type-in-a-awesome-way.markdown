---
layout: post
title: "Using map method in a incredibly elegant way"
date: 2012-09-22 01:38
comments: true
categories: [tips, ruby, metaprogramming]
---

今天在写最近的项目的时候，遇到一个问题，本来是想等这个项目完了之后，把学到的一些一起写出来，不过觉得那时应该会工作量比较大，也比较拥挤。
问题是，需要把多个参数转换为一个integer，然后再-1，如下：
```
foo1, foo2, too3 ... = foo1.to_i - 1, foo2.to_i - 1, foo3.to_i -1 ...
```
如果变量一多，就会很麻烦，要重复写好多（我那里是8个，用了三大行才写下），就去ruby-china上求助，然后很快得到回复，然后得到了一种很漂亮的方法
```
[foo1, foo2, foo3].map(&:to_i).map(&:pred)
```
这真算是种一看就亮了的方法吧，**awesome**
##map
map的话，确实因为ruby用的不熟，还没想到去用，而且也没考虑要把他们作为一个数组来对待，这其实就是说明一堆没有规律的数据堆在一起，不管操作还是干嘛都很不方便。
所以这里用数组组织起来确实是很合适的，当然，这样直接赋值后的话，还并没有改变那几个变量本身，不过还是可以把结果保存在一个数组中来用的。
##&:symbol
最cool的应该是这个吧，看上去是挺容易看懂，就是把每次迭代器的参数对它自身调用`to_i`方法，并连续调用map。
不过详细解释的话，还是去搜了一下,就在[stackoverflow的一个问题](http://stackoverflow.com/questions/1217088/what-does-mapname-mean-in-ruby)里看到了相关话题。
那个被采用的回答里提到
>It's shorthand for tags.map(&:name.to_proc).join(' ')
>
>If foo is an object with a to_proc method, then you can pass it to a method as &foo, which will call foo.to_proc and use that as the method's block.

<!-- More -->

然后参考这个网站，著名的[Dave Thomas的网站](http://pragdave.pragprog.com/pragdave/2005/11/symbolto_proc.html)

>When you say names.map(&xxx), you’re telling Ruby to pass the Proc object in xxx to map as a block. 
>
>If xxx isn’t already a Proc object, Ruby tries to coerce it into one by sending it a to_proc message.

这段就是说**一个对象`bar`，如果本身是一个`Proc`，就把它传到`map`中，如果有一个`to_proc`方法，那么就调用`bar.to_proc`，并把这个转换成的`Proc`对象传入`map`中**。
##Symbol and Proc
去查一下ruby的api关于[Symbol类的to_proc方法](http://www.ruby-doc.org/core-1.9.3/Symbol.html#method-i-to_proc)
就会知道对于一个Symbol的对象，比如这里的`:to_i`前边加上`&`，其实就是调用了它的`to_proc`方法而成为Proc的对象，并传给`map`这个迭代器。
`Symbol#to_proc`的代码像这样：
```
def to_proc
  proc { |obj, *args| obj.send(self, *args) }
end
```
上边的代码中`send`是`Object`类中定义的，就像这样`send(symbol [, args...]) → obj`，其实就是调用了obj的名字叫“symbol”的方法。
在把上述这个proc对象传到map迭代器之后，每次迭代的数组中的一个值就相当于是`to_proc`方法中的`obj`对象，然后就对它调用`to_i`方法，就可以了。

##Summary
这应该已经涉及到了元编程的动态调用方法和一些闭包的内容了，真是博大精深啊。
正如Dave Thomas那篇文章最后说的

>It’s an incredibly elegant use of coercion and of closures.

PS：其实那篇是很久前写的，他提到了[Ruby Extensions Project](http://extensions.rubyforge.org/rdoc/index.html)
不过现在ruby里已经把Symbol的这个方法添加进去了，而那本《Programming ruby》（379页）里也有和这篇博文差不多的一个叙述。
这本书里又说到了，这种方式可能会造成效率的下降，不过一般的程序，还是可以用的
