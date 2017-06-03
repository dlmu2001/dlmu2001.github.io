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
![](/assets/media/classloader_hierarchy.gif) 
   
   图2 Java中的ClassLoader层次结构

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
　    


## 1.6 Android APK打包过程

![](/assets/media/apk.png) 

  图3 apk打包过程 

## 1.7 Android平台的ClassLoader

* 如1.6图３所示，Android平台打包.class的时候，生成的不是.jar，而是.dex，它是Android平台的一个优化，Android dx程序会对所有.class进行优化合并，不同.class重复的东西只要保留一份.

* 如果应用不进行分包处理，也没有动态加载，一个APK只会有一个dex文件

* Android中类加载器有BootClassLoader,URLClassLoader, PathClassLoader,DexClassLoader,BaseDexClassLoader,等都最终继承自java.lang.ClassLoader
  
  - BootClassLoader:是Android平台上所有ClassLoader的最终parent,这个内部类是包内可见,开发者一般没法使用
  - BaseDexClassLoader:基于Dex的ClassLoader实现，PathClassLoader和DexClassLoader的实现。这个类比较重要，后面再展开。
  - PathClassLoader:Android系统和应用的class loader
  - DexClassLoader：DexClassLoader支持加载APK、DEX和JAR，也可以从SD卡进行加载。他会将APK,JAR处理成DEX交给虚拟机。通常我们自己定制ClassLoader都继承自这个类。

![](/assets/media/android_classloader.png) 

   图4 android平台的class loader

## 1.8 BaseDexLoader
* 构造函数参数dexPath，传入dex列表，根据这个列表，load每个dex,形成一个DexFile和dex名字一一对应的Element列表DexPathList
* 一个BaseDexClassLoader可以包含多个dex文件，每个dex文件是一个Element，多个dex文件排列成一个有序的数组dexElements，当找类的时候，会按顺序遍历dex文件，然后从当前遍历的dex文件中找类，如果找类则返回，如果找不到从下一个dex文件继续查找。
* 如果在不同的dex中有相同的类存在，那么会优先选择排在前面的dex文件的类
* DexFile::loadClassBinaryName最终会调用native函数

![](/assets/media/basedexclassloader1.png)

   图5 BaseDexClassLoader类

 
![](/assets/media/BaseDexClassLoader2.png)
   
   图6 DexPathList的findClass实现

## 1.9 dalvik 和Art
* dalvik和Art都是Java Runtime
* Android 5.0后,Art取代了Dalvik，ART能够把应用程序的字节码转换为机器码，是Android所使用 的一种新的虚拟机。它与Dalvik的主要不同在于：Dalvik采用的是JIT技术，字节码都需要通过即时编译器（just in time ，JIT）转换为机器码，这会拖慢应用的运行效率，而ART采用Ahead-of-time（AOT）技术，应用在第一次安装的时候，字节码就会预先编译成机器码，这个过程叫做预编译。ART同时也改善了性能、垃圾回收（Garbage Collection）、应用程序除错以及性能分析。但是，运行时内存占用空间较少同样意味着编译二进制需要更高的存储。
* ART模式相比原来的Dalvik，会在安装APK的时候，使用Android系统自带的dex2oat工具把APK里面的.dex文件转化成OAT文件，OAT文件是一种Android私有ELF文件格式，它不仅包含有从DEX文件翻译而来的本地机器指令，还包含有原来的DEX文件内容。

# 2.0 QQ空间热补丁修复

## 2.1 基本思路

* 原理主要来自1.8所述，灵感来源于Android Dex分包方案
* 将要修复的类打包成一个patch.dex， 假设原来的ａｐｋ包含的classes.dex。
* 通过获取到当前应用的Classloader(即为BaseDexClassloade),反射得到ClassLoader的dexPathList，反射调用pathList的dexElements方法把patch.dex转化为Element[],插入到原来的dexPathList的最前面，这样子patch.dex的类会以最高优先级加载。

![](/assets/media/qzone.png)
   
   图7 QQ空间热修复基本原理 

## 2.2 插桩

### 2.2.1 问题
* 假定ModuleManager在patch.dex，而QzoneActivityManager在原来的classes.dex，如果ModuleManager调用QzoneActivityManager，会出错。出错的原因是在解析类的时候，会校验饮用者和被引用者的dex是否相同。这个校验无法通过。

* ＱＱ空间团队观察到,拆分Dex(MulDex)的实现不会进行这种校验，原因是拆分Dex里面，类没有被打上CLASSISPREVERIFIED标志。如果引用者(ModuleManager)被打上CLASSISPREVERIFIED标志，就会进行校验。


* 如果static方法，构造函数，private方法中直接引用到的类（第一层级关系，不会进行递归搜索）和clazz都在同一个dex中的话，那么这个类就会被打上CLASS_ISPREVERIFIED标志。因此，如果在构造函数中引用到其它dex的类，这个类不会被打上标志。

### 2.2.2解决
* 往所有类的构造函数插入一段代码，调用System.out.println(AntiLazyLoad.class);去调用来自另外一个dex的类AntiLazyLoad

* AntiLazyLoad单独打包成一个hack.dex

* 往所有类的构造函数插入代码的工作由Gradle插件完成

### 2.3 方案分析
* 优点：开发透明，简单，应用补丁成功率高;支持资源替换

* 缺点:Dalvik平台，对启动耗时有一定影响。Art平台性能没有影响，但是如果修改类涉及到修改类变量或者方法(static)，可能会导致内存地址错乱问题，需要将修改了变量、方法以及接口的类的父类以及调用这个类的所有类都加入到补丁包中。这可能会带来补丁包大小的急剧增加。不支持即时生效。

# 3. AndFix热修复框架

## 3.1 原理
* 从1.4的图可以看出，类的方法在Method Area，一个类只会有一个，会有一个指针地址

* AndFix的原理就是方法的替换，用native方法把有bug的方法替换成补丁文件中的方法,相当于进行了native hook
        
        //java方法
        void replaceMethod(ClassLoader classLoader, String clz,
            String bugMethodName, Method method)
        //jni方法,每个版本需要一个jni方法
        void replace_5_0(JNIEnv* env, jobject src, jobject dest);

* 方法替换过程

![](/assets/media/andfix.png)
   
   图８　AndFix方法替换过程

* AndFix补丁包和原包的关系

![](/assets/media/andfix2.png)
   
   图9　AndFix补丁包

* AndFix方案开发流程

![](/assets/media/andfix3.png)
   
   图10　AndFix开发流程

## 4.2 方案分析

* 优点:即时生效，不需要重启应用。补丁较小

* 缺点:兼容性问题不佳；开发不透明；应用场景受限(比如由于它并没有整体替换class, 而field在class中的相对地址在class加载时已确定，所以AndFix无法支持新增或者删除filed的情况）。

* AndFix也就是现在的阿里百川HotFix，定位于紧急bug修复。手机淘宝因为在插件化上做了很多尝试(Atlas)，同时正在热推weex，对热修复场景的需求反而不那么强。因为多个dex，所以手淘不太适合使用QQ空间或者微信tinker的方案。

* 手淘已经解决了兼容的问题，称为Sophfix，但是没有开源,方案还绑定后台

# 5. 微信热更新Tinker

## 5.1 方案原理
* 全量更新Dex
* 通过反射操作得到PathClassLoader的DexPatchList,反射调用patchlist的makeDexElements()方法把本地的dex文件直接替换到Element[]数组中去，达到修复的目的（图12)
* 补丁包为差分包
* 简单来说，在编译时通过新旧两个Dex生成差异path.dex。在运行时，将差异patch.dex重新跟原始安装包的旧Dex还原为新的Dex。还原过程可能比较耗费时间与内存，单独放在一个后台进程:patch中
* 为了补丁包尽量的小，微信自研了DexDiff算法，它深度利用Dex的格式来减少差异的大小
* 生成patch的过程由Gradle脚本完成，对开发透明

![](/assets/media/tinker.png)
   
   图11 Tinker全量更新　

![](/assets/media/tinker2.png)
   
   图12 Tinker原理　

![](/assets/media/tinker3.png)
   
   图13 Tinker流程图　

* so的更新思路类似class，ClassLoader的DexPathList有so的查找逻辑，将补丁包的so路径插入到so查找路径的最前面。麻烦在于这里Android各个版本不一致，需要做版本兼容，而且要考虑这个so是否已经加载过,ＡBI的选择也是个问题。

![](/assets/media/hotfix_so.png)
   
   图14 so查找逻辑　

![](/assets/media/hotfix_so2.png)
   
   图15 系统load so流程　

* 资源的更新：资源的更新略复杂，通过调用addAssetPath将patch的资源加到新建的AssetManager对象中，然后将内存中所有Resources对象中的AssetManager对象替换为新建的AssetManager对象，具体可以参考[Android 热修复方案Tinker(四) 资源补丁加载](http://blog.csdn.net/l2show/article/details/53454933)

## 5.2 方案分析
* 优点：应用场景非常广泛；兼容性和稳定性比较高;
* 缺点：占用磁盘空间(大约是你修改Dex数量的1.5倍(dexopt与dex压缩成jar)的大小);一个额外的合成过程；不支持即时生效;Do not support some Samsung models with os version android-21

# 6.补丁方案比较
* 由于ＱＱ空间没有开源，这里使用了和qq空间思路一样的nuwa，可以认为是qq空间的方案

![](/assets/media/hotfix_compare.png)

   图16 各补丁方案比较

# 7.其它
* 补丁生效时机
* 补丁分发
* 补丁下载
* 补丁监控


## 参考
1. [Internals of Java Class Loading](http://www.onjava.com/pub/a/onjava/2005/01/26/classloading.html?page=1)

2. [Java Virtual Machine Specification](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-5.html)

3. [安卓App热补丁动态修复技术介绍](https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a)

4. [Inside Class Loaders](http://www.onjava.com/pub/a/onjava/2003/11/12/classloader.html)

5. [java虚拟机工作原理详解](http://www.cnblogs.com/miss510/p/4975805.html)

6. [Android Apk打包过程概述](http://blog.csdn.net/jason0539/article/details/44917745)
