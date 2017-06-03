---
layout: post
title: Android 热更新机制研究 
categories: [Android]
tags: [动态修复]
description: 热更新的基础,以及各种热更新实现机制分析
---

tomorrow.cyz@gmail.com

# 1.理论基础　

## 1.1 class
* 大部分Java 应用程序用.class来表示一个类，一般情况下,.class的文件名和它的package/class　名存在一一对应关系。

* 本质上，Class代表了一段code,load　class相当于把这段code加载到内存。类实例则是数据对象。
”A class represents the code to be executed, whereas data represents the state associated 
 with that code.State can change; code generally does not.“

* javac编译指令将.java的source code编译生成`java bytecode`，而不是机器码。也就是说javac只是前端编译器，在运行时再进行解释执行。对于热点代码（频繁执行），`运行时`会做JIT(Just In Time Compiler)，JIT通常由虚拟机实现。.class就是java bytecode。

* `A combination of the class type and effective class loader.`, 在这里，class type指`The fully qualified class name (package plus class name).`

## 1.2 Java类的执行过程
* 1.2.1 Loading
  - 通过类型的完全限定名，产生一个代表该类型的二进制数据流
  - 构创建一个表示该类型的java.lang.Class类的实例。
* 1.2.2 Linking
  - 确认类型符合Java语言的语义，检查各个类之间的二进制兼容性(比如final的类不用拥有子类等)，另外还需要进行符号引用的验证；
  - 准备，Java虚拟机为类变量分配内存，设置默认初始值；
  - 解析(可选的) ，在类型的常量池中寻找类，接口，字段和方法的符号引用，把这些符号引用替换成直接引用的过程
* 1.2.3 Initializing
  - 类和接口初始化`<clinit>`。如下代码code block a就是由`<clinit>`调用。

        class A{
            static {
            //code block a
            }
        }

## 1.3 什么时候触发类加载(Loading)
* 实例化一个类
* 调用某个类/接口的静态方法或者静态字段
* 类的预加载

## 1.4 ClassLoader
* ClassLoader完成1.2.1中的类加载，简单地说，就是将一个完全限定名转化成java bytecode。

* 代码抽象的1.2.1的过程

        Class r = loadClass(String className, boolean resolveIt); 

* ClassLoader将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在堆区创建一个java.lang.Class对象，用来封装类在方法区内的数据结构。类的加载的最终产品是位于堆区中的Class对象，Class对象封装了类在方法区内的数据结构，并且向Java程序员提供了访问方法区内的数据结构的接口。
![](/assets/media/java_runtim_data_areas.png) 

    图1 Java运行时数据区

## 1.5 Java中的ClassLoader层级以及加载机制
<p align="center">
![](/assets/media/classloader_hierarchy.gif) 

图2 Java中的ClassLoader层次结构
</p>
* 1.5.1 层次结构
  - Bootstrap Class Loader:当运行java虚拟机时，这个类加载器被创建，它加载一些基本的java API，包括Object这个类。这个类加载器不是用java语言写的，而是用C/C++写的。
  - Extension Class Loader:加载JAVA_HOME/lib/ext目录下的或-Djava.ext.dirs指定目录下的jar包
  - System Class Loader:加载classpath或者-Djava.class.path指定目录下的类或jar包
  - User-Defined Class Loader:开发人员通过拓展ClassLoader类定义的自定义加载器，加载程序员定义的一些类

* 1.5.2 Java类加载机制

  Java使用一个”代理模型“来加载类，这个模型的基础是每个ClassLoader都有一个parent ClassLoader。加载类的时候，首先委托父类进行加载，如果父类无法加载，自己再进行加载。

  步骤如下

    - java.lang.ClassLoader的构造器接受一个parent的参数，来指定parent ClassLoader，如果没有指定，就使用缺省的系统类加载器作为parent ClassLoader
    - ClassLoader的loadClass方法依次执行如下动作 a.如果类已经被加载过，返回　b. 委托parent加载这个类，能够加载到就返回 c.调用本加载类的findClass来加载
    - 安全起见，自定义ClassLoader通常实现findClass方法，而不是重载loadClass方法破坏委托模型。
       
            protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {
                Class<?> clazz = findLoadedClass(className);
                if (clazz == null) {
                    ClassNotFoundException suppressed = null;
                    try {
                        clazz = parent.loadClass(className, false);
                    } catch (ClassNotFoundException e) {
                        suppressed = e;
                    }fei
                    if (clazz == null) {
                        try {
                            clazz = findClass(className);
                        } catch (ClassNotFoundException e) {
                            e.addSuppressed(suppressed);
                            throw e;
                        }
                    }
                }
                return clazz;
            }
            protected Class<?> findClass(String name) throws ClassNotFoundException {
                return Class.classForName(name, false, null);
            }
　    


