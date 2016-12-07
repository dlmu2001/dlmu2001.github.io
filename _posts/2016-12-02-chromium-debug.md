---
layout: post
title: chromium的debug手段
categories: [chromium]
tags: [debug]
description: chromium的主要调试手段，包括Java和native code，比如打log，打印堆栈，gdb调试，breadkpad等 
---

tomorrow.cyz@gmail.com

Chromium代码常见调试手段如下

## Java code 
   - log打印
        * 对于Android source tree的代码，使用android.util.Log
        * 对于Chromium source tree的代码,使用org.chromium.base.log,用法和	android.util.Log一样
    
   - 打印调用堆栈
        * Log.d(TAG,Log.getStackTraceString(new Throwable()));
    
   - 调用跟踪
        * TraceEvent.begin/TraceEvent.end 
    
   - 单步调试
        * 使用ide（Android studio)

## chromium native c++
   - log打印
        * LOG/DLOG/VLOG
        * LOG(INFO) << "Found "<<num_cookies<<" cookies";
        * 四种日志级别:FATAL,ERROR,WARNING,INFO。FATAL会在日志输出后自动引发一个crash
        * DLOG只在DEBUG版本生效
        * VLOG可以用verbose级别来控制是否生效,当启动开关--v=1时，VLOG 1以上级别生效
        * 还有LOG_IF提供额外判断条件
        
  - 打印调用堆栈
           #include "base/debug/stack_trace.h"
                base::debug::StackTrace(); 
                           
  - 调用跟踪
    * 通过TraceEvent.setATraceEnabled来控制调用跟踪是否打开
    * TRACE_EVENT0(catogary,name);如TRACE_EVENT0("startup","ContentMainRunnerImpl::Initialize");
         - 通常在App实现里面，通过CommandLine或者SystemProperties来动态设置，比如android_webview_test_shell，可以echo "chrome --enable-atrace">/data/local/tmp/android-webview-command-line" 来打开Android trace
         - Android Systrace的抓取和分析:https://developer.android.com/studio/profile/systrace.html#app-trace"
        
   - 使用gdb进行单步调试
       * device端
           - 如果没有gdbserver，拷贝进去
                * gdbserver :6000 --attach PID     
                * PID是要调试的进程，可以通过ps|grep获取
        * host端
                * adb forward tcp:6000 tcp:6000
                * arm-linux-androideabi-gdb
                * target remote :6000
                * set solib-search-path $SOPATH
                *  SOPATH是带符号的so所在的路径
            
## jni
   - log打印:__android_log_print
  
   - 打印调用堆栈
 	#include &lt;utils/CallStack.h&gt;
 	CallStack stack;
        stack.update();
        stack.dump();    

## breakpad分析crash堆栈
 - 在src目录下执行命令:ninja -C out/Release microdump_stackwalk生成microdump_stackwalk
 
 - 调用microdump_stackwalk获取crash堆栈，其中logcat.log是android logcat的output
       * microdump_stackwalk logcat.log
 
 - 使用addr2line命令分析symbol,其中libchromium.so是带symbol的so库文件，-f 后面的0xxxxxxxxx是堆栈地址
      * addr2line -e libchromium.so -f 0xxxxxx
