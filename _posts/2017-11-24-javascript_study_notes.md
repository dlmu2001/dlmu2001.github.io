---
layout: post
title:  javascript学习笔记 
categories: [fontend]
tags: [语言]
description: 一个Android开发人员的javascript学习笔记 
---

tomorrow.cyz@gmail.com 


## 1.1 javascipt是一门灵活的动态语言

一般情况下，动态语言都比静态语言灵活，而javascript又是动态语言里面比较灵活的。
这种灵活性让人印象深刻。下面是一些例子


* javascript以key-value的方式来定义一个对象(有点像java里面的map，或者oc里面的dictonary)，
它允许在对象声明后任何时刻新增/删除属性，新增/删除/更新接口，这在静态语言里面是比较难以想象
的。另外这种key-value的定义方式，在赋值的时候就非常的方便，比如解构赋值。天然适合和Json结合。


* javascript可以通过require(表达式)实现类似的运行时require，这个特性对多平台上实现跨平台非常
有帮助。java的import或者c++里面的include都无法提供
这种属性，在需要跨平台的时候，往往需要在工程级别通过python写code generator来实现，比较麻烦。


* 弱类型。javascript是弱类型变量，解释器运行时检查数据类型。传统的oop语言如java/c++都采用强
类型检查，所以所有的变量在使用之前必须做声明，而声明后变量类型就固定下来了，这一点javascript
灵活很多。虽然ES6也开始采用了类似的声明方式，但是本质上javascript还是弱类型的。比如java/C++为
了泛型，还需要引入模板的概念，而javascript，根本不需要泛型（或者说缺省就是泛型的）。除了泛型，
javascript在支持多种参数，缺省参数，多种返回值各种方面的灵活性让人惊叹。当然，弱类型是一把双
刃剑，它提供了灵活性的同时，安全性上会打折扣。比如静态语言的契约/接口通过类型检查，在编译期
就可以暴露错误，而动态语言，则只有在运行时猜暴露错误。接口的约定就更依赖于注释等不可靠、非
强制性的东西。


* javascript可以改变/增加原型的方法。比如说有个框架/平台提供的类Log，这个类Log在全App内广泛使
用，现在想改变Log.D的行为，如果是java，只能通过集成Log，然后App内全局替换Log为集成类来实现，工
作量比较大，而且还有像String这种final类不能继承的问题。javascript直接修改Log类的原型方法就可以。
OC里面通过Categories可以实现往基类添加新成员，通过Posing来修改基类的方法，影响范围在App内部，
不清楚javascript这个影响范围是如何控制的？在一个沙盒里面?

## 1.2 弱化类的概念

javascript弱化了类的概念，ES6的class是一个语法糖。它的继承是原型继承。

<br/>
它本质上是没有权限访问控制的特性，比如说没有支持私有方法这种在面向对象编程里面很重要的特性，要
通过模拟来实现。

<br/>
它本质上既不支持多重继承，也不支持接口。

<br/>
通过ES6 extend实现继承的方式，它的实例变量就有点特别，在继承类中不能通过super.属性的方式访问，
而外部可以通过父类对象.属性的方式访问。

<br/>
本质上，javascript把一切当成对象，对象和类是不同的概念。

<br/>
函数也是对象，而且javascript还存在顶层对象一说。这两个特性给this这个关键字带来了不确定性。比如
说可以把一个类的接口赋值给一个变量，这个时候，接口里面如果有this关键字，通过这个变量调用接口和
通过实例调用接口，this就有不同的意义。这就有了this的绑定(call/bind/apply)。

## 1.3 事件驱动

一般情况下，语言很少提供事件驱动的特性，大部分情况下，事件驱动依赖于平台，而不是
语言本身。从这一点上看，javascript拥有了一部分平台特性，这个平台就是V8或者其它core。

<br/>
## 1.4 单线程模型

javascript使用了单线程模型，这一点和大部分原生的系统不一样。这意味着在大部分场合（
特别是io），不需要考虑线程安全的问题。以此带来的副作用，就是javascript里面大量地使用callback。
在嵌套callback的情况下，代码可读性下降。因此javascript引入了很多方案来帮助异步编程，如generator
/promise/async等等。

## 1.5 其它

* 闭包很像加了命名空间的全局变量，或者像c语言函数里面的static变量加了命名空间

* Android里面现在用的很多的rxjava可以通过promise/asnc await/co等实现

* Generator在回复执行的时候还可以通过给next传参数调整函数行为
