---
layout: post
title: "JS GoodPart: Object"
date: 2012-09-17 03:03
comments: true
categories: [javascript, goodpart, book, note]
---
javsctipt 简单类型包括数字、字符串、boolean、null和undefined，其余都是对象（**和Ruby不同**）

是可变的键控集合（keyed collections）

##对象字面量

一个对象字面量就是在一对花括号中的0或多个“键/值”对

**对象是属性的容器**，每个属性有**名字和值**，属性名可选择是否用**双引号**括起来

##检索

1. 【】+字符串
2. foo.bar （优先）

不存在时返回**undefined**

* 用||来赋默认值
* 用&&来避免undefined错误

```
var middle = stooge["middle-name"] || "(none)";
var status = flight.status || "unknown";
flight.equipment // undefined
flight.equipment.model // throw "TypeError"
flight.equipment && flight.equipment.model // undefined
```
<!-- More -->
##赋值

####***PS： 如果属性不存在，该属性就被加入对象到属性中去***

##引用

对象通过引用来传递，***他们永远不会被拷贝***

```
var x = stooge;
x.nickname = 'Curly';
var nick = stooge.nickname;
// nick is 'Curly' because x and stooge
// are references to the same object
var a = {}, b = {}, c = {};
// a, b, and c each refer to a
// different empty object
a = b = c = {};
// a, b, and c all refer to
// the same empty object
```

##原型（prototype）

每个对象都链接到一个原型，并从中继承属性，通过字面量创建的原型为`Object.prototype`

#### 原型链 委托（delegation）

 只有检索时才会用到。沿原型链寻找，直到到达`Object.prototype`，否则结果为undefined

这是种动态关系，当添加新属性到原型中，该属性会对此对象可见。

##反射Reflection

用typeof来得到属性的类型

***PS：对原型链的属性也会有作用***

两个方法处理不需要的属性

1. 让程序检查并去掉函数的类型，因为一般来说做反射的目标是数据。

2. 用`hasOwnProperty()`方法，不检查原型链的。

##枚举Enumeration

用for in来遍历，这会列出所有属性，因此有必要过滤掉不需要的。

```
var name;
for (name in another_stooge) {
if (typeof another_stooge[name] !== 'function') {
document.writeln(name + ': ' + another_stooge[name]);
}
}
```

***属性名出现的顺序是不确定的！！如果要确保以特定顺序出现，最好的方法就是完全避免使用for in，而是创建一个数组***

```
var i;
var properties = [
'first-name',
'middle-name',
'last-name',
'profession'
];
for (i = 0; i < properties.length; i += 1) {
document.writeln(properties[i] + ': ' +
another_stooge[properties[i]]);
}
}
```

##删除

delete可以删除对象中的属性，而不会删除prototype链中的。

**PS：删除对象中的属性之后，可能会让原型链中的属性浮现出来**

##减少全局变量污染Global Abatement

javascript可以很随意定义全局变量，**不过它削弱了程序的灵活性，应该避免**

最小化使用全局变量的一个方法是在应用中只创建唯一一个全局变量，此变量就变成应用的容器

```
var MYAPP = {};
MYAPP.stooge = {
"first-name": "Joe",
"last-name": "Howard"
};
MYAPP.flight = {
airline: "Oceanic",
number: 815,
departure: {
IATA: "SYD",
time: "2004-09-22 14:55",
city: "Sydney"
},
arrival: {
IATA: "LAX",
time: "2004-09-23 10:42",
city: "Los Angeles"
}
};
```

可以有效地把多个全局变量整理在一个命名空间下（这里`MYAPP`就起到命名空间的作用），可以更容易和其他程序整合，也更易读。
