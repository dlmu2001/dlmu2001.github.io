---
layout: post
title: NDK开发 
categories: [Android]
tags: [NDK]
description: 如何利用NDK为App开发Native程序。team内部培训 
---

tomorrow.cyz@gmail.com

# 1.名词解释
Native:Android应用程序缺省情况下使用Java开发，编译成byte code,最后在target上需要由虚拟机转化成机器语言。Java被设计成一种跨
平台的语言。与之相比，Native程序一般以C/C++语言编写，会直接编译成对应平台的机器语言。

JNI:Java Native Interface,是一种编程框架，通过JNI，运行在JVM上的Java代码可以调用native程序，也可以被native程序调用。

NDK:The Native Development Kit,Android提供了一套可以使用C/C++的开发环境和工具，NDK提供了平台库用于管理Native Activities
和物理设备，比如Sensor。

ABI:application binary interface。应用程序二进制接口，在Android上，使用的是EABI(Embed application binary interface)。是
描述可链接目标代码、库目标代码、可执行文件映像，如何连接，执行和调试，以及目标代码生成过程。同一份代码要在不同的EAＢＩ下
工作，必须生成对应的EAＢＩ的binary。

# 2.应用程序上为什么使用Native开发
* 重用开源库或者其它开发者／平台的C/C++ library

* 代码更难被反编，可以用来放置安全相关的代码，提高攻击成本

* 对性能有要求的地方,比如游戏或者物理仿真

# 3. HelloWorld

* native部分代码         
        
        native_util/
        └── jni
            ├── Android.mk
            ├── Application.mk
            ├── entry.cc
            ├── math.c
            └── math.h

        //Android.mk
        LOCAL_PATH := $(call my-dir)
        
        include $(CLEAR_VARS)
        
        LOCAL_MODULE    := native_util
        LOCAL_SRC_FILES :=  \
        	entry.cc \
        	math.c \
        
        LOCAL_LDLIBS := -llog
        
        include $(BUILD_SHARED_LIBRARY)


        //Application.mk
        APP_ABI := all

        //entry.cc
        #include <jni.h>
        #define DEBUG 1
        
        #if DEBUG
        #include <android/log.h>
          #define  LOG(x...)  __android_log_print(ANDROID_LOG_INFO,"JniExample native",x)
        #else
          #define  LOG(...)  do {} while (0)
        #endif
        extern "C" {
          #include "math.h"
        }
        #define NELEM(x) ((int) (sizeof(x) / sizeof((x)[0])))
        
        jint mathAdd(JNIEnv* env,jclass,jint x,jint y){
          jint result= add(x,y);
          LOG("result=%d",result);
          return result;
        }
        
        
        const char kClassName[] = "org/linux/jniexample/utils/NativeUtil";
        const JNINativeMethod kJniMethods[]={
                {"nativeAdd","(II)I",
                        (void*)mathAdd},
        };
        
        JNIEXPORT jint JNI_OnLoad(JavaVM* vm, void* reserved) {
          JNIEnv* env = 0;
          jint ret = vm->AttachCurrentThread(&env, 0);
          LOG("JNI_OnLoad");
          if(ret!=JNI_OK){
            LOG("JNI_OnLoad failed");
          }
          jclass clazz = env->FindClass(kClassName);
          int res=env->RegisterNatives(clazz,kJniMethods,NELEM(kJniMethods));
          if(res<0){
              LOG("RegisterNatives failed,res=%d",res);
          }
          return JNI_VERSION_1_4;
        }

        //math.h
        int add(int x,int y);

       //math.c
       #include "math.h"

       int add(int x,int y){
           return x+y;
       }
        
* java部分代码

        package org.linux.jniexample.utils;        
        public class NativeUtil {
            static {
                System.loadLibrary("native_util");
            }
            public static int add(int x,int y){
                return nativeAdd(x,y);
            }
        
            public static native int nativeAdd(int x,int y);
        }

* 说明
    - NDK的代码可以使用两种编译方式，这里用的是ndk-build，另外一种是CMake。其中ndk-build既支持Android Studio，也支持命令行编译。所以我一般采用ndk-build
    
    - Android Studio集成ndk-build，会自动编译native代码，生成对应的so，拷贝到对应的工程目录下
    
    - ndk-build编译的方式，需要有Android.mk和Application.mk两个文件
    
    - Android.mk定义模块名(LOCAL_MODULE)，要编译的源文件，编译flag和要链接的lib,这个mk文件和aosp的mk文件类似,[这里](https://developer.android.com/ndk/guides/android_mk.html)有详细介绍
    
    - Application.mk一般用于定义toolchain，支持的EABI，[这里](https://developer.android.com/ndk/guides/application_mk.html)有详细介绍。
    
    - 一般利用Java类的static代码块来加载library，这个loadLibrary的调用首先会来到JNI_OnLoad接口
    
    - 在Java代码中声明Jni接口
        
       public static native int nativeAdd(int x,int y);

    - 如果没有在JNI_OnLoad做接口映射，默认上面Java声明的jni的函数为Java_org_linux_jniexample_utils_NativeUtil_nativeAdd，格式是Java_PkgName_ClassName_NativeMethodName，非常容易出错。
所以一般做映射，通过RegisterNatives接口来映射Java类里面声明的jni方法和实际的jni方法。

    - Android Studio里面可以直接拷贝native_util到对应的目录，然后选择Project模式，对要关联native程序的module右击，在菜单link C++ Project with gradle中将native代码加入工程。
　　
    - native工程加入Android Studio以后，Studio会要求下载NDK和LLDB，其中LLDB是调试程序，Studio用它来调试native代码

# 4.JNI数据类型
* 数据类型转换是jni变成比较重要的一部分工作，非常枯燥，容易出错

* JNI数据类型包括基本类型和引用类型

* 基本的数据类型可以直接转换。

                
        JavaType	JNIType 	C/C++Type	    Size
        Boolean     jboolean	unsigned char	unsigned 8 bits
        Byte        jbyte   	char	        singned 8 bits
        Char        jchar       unsigned short	unsigned 16 bits
        Short       jshort      short           signed 16 bits
        int         jint	    int 	        signed 32 bits
        Long        jlong       long long       signed 64 bits
        Float       jfloat	    float           32 bits
        Double      jdouble	    double	        64 bits

*  注意jni没有unsigned int这种类型，需要使用jlong，对应的是long long

* JNI中引用类型主要有类、对象和数组，对应的关系如下
        

        JNI 类型	Java引用类型	描述
        jobject	Object	    Object类型
        jclass	Class	    Class类型
        jstring	String	    String类型
        jobjectArray	Object[]	对象数组
        jbooleanArray	boolean[]	boolean数组
        jbyteArray	byte[]	    byte数组
        jcharArray	char[]	    char数组
        jshortArray	short[]	    short数组
        jintArray	int[]	    int数组
        jlongArray	long[]	    long数组
        jfloatArray	float[]	    float数组
        jdoubleArray	double[]    double数组
        jthrowable	Throwable	Throwable

# 5.JNI类型签名
* 接口映射的时候需要使用JNI类型签名标识一个特定的Java类型，这个类型可以是类和方法，也可以是基本数据类型

* 类型签名
        
        类的签名采用”L+包名+类名+;”标识，包名中将.替换为/即可。
        比如String类的签名：
        Ljava/lang/String;
        注意末尾的;属于签名的一部分。
        再比如Android中Context类的签名：
        Landroid/content/Context;

* 基本数据签名采用一系列大写字母来标识
        
        Java类型    签名
        boolean     Z
        byte        B
        char        C
        short       S
        int         I
        long        J
        float       F
        double      D
        void        V

* 数组类型签名
        
        数组的类型签名比起类类型和基本数据类型的要稍微复杂一点，不过还是很好理解的。对于数组来说，它的签名为[+类型签名，举例说明：
        String[] 数组类型对应的签名：
        [Ljava/lang/String;
        可以发现，就是在String的类签名前加了个[
        同理基本数据类型签名int[]的签名：
        [I
        注意这里基本类型后面是不带分号的。
        那么多维数组呢？可以类推，int[][] 的签名为[[I ，而String[][]的签名为[[Ljava/lang/String; 

* 方法签名
        
        在JNI中会经常需要在C/C++代码中调用Java的函数，这时候就会用到方法的签名。方法的签名为(+参数类型签名+)+返回值类型签名，比如：
        方法： boolean login(String username,String password)的方法签名如下：
        (Ljava/lang/String;Ljava/lang/String)B 

# 6.JavaVM和JNIEnv
* JavaVM有点类似虚拟机实例，在JNI_OnLoad的时候会传进来，Android每个线程只允许有一个JavaVM,一般在JNI_OnLoad的时候来attach 线程

* 所有的JNI函数，即使在声明的时候没有任何参数，也会带两个默认参数,JNIEnv和jclass

* 通过JNIEnv可以访问系统提供的JNI接口(FindClass,GetFieldID,NewStringUTF等)，这里注意JNI的c实现和c++实现略有不同，我们的helloworld中使用c++编写，
就直接使用env->，如果是ｃ语言，则这样用(*env)->


* 通过JNIEnv，可以实现在jni代码中反射到所有的java类，从而实现JNI中调用java代码
        
        //NativeUtil.java
        public static void javaFunc() {    
            Log.d("NativeUtil","Here's java function");
        }
        
        //entry.cc
        static void static_javaFunc(JNIEnv *env){
              jclass clazz = env->FindClass("org/linux/jniexample/utils/NativeUtil");
              jmethodID mid = env->GetStaticMethodID(clazz, "javaFunc", "()V");
              env->CallStaticVoidMethod(clazz, mid);
        }
        //调用
        static_javaFunc(env);

# 7. 其它
* 认真阅读[JNI Tips](https://developer.android.com/training/articles/perf-jni.html)

# 参考
* 1.[Add C and C++ Code to Your Project](https://developer.android.com/studio/projects/add-native-code.html) 

* 2.[add-native-code](https://developer.android.com/studio/projects/add-native-code.html#link-gradle)    

* 3.[JNI Tips](https://developer.android.com/training/articles/perf-jni.html)

* 4.[Android JNI&NDK编程小结及建议](http://www.cnblogs.com/android-blogs/p/5782671.html)   

        


