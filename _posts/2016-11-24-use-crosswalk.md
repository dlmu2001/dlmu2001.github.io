---
layout: post
title: Android上使用Crosswalk，实现app内核定制 
categories: [Android]
tags: [Chromium]
fullview: true
---

# 什么是CrossWalk
   CrossWalk是基于Chromium内核的一款HTML应用运行环境，用户可以在上面搭建HTML应用。跨平台开发环境Cordova现在就将CrossWalk当做自己的运行环境。
   使用CrossWalk，可以直接开发Web App，在Android上，也可以将他当做一个Embed HTML Runtime，替代系统WebView。

# 为什么使用CrossWalk
   很多公司使用系统的WebView桥接来实现Native和H5的混合开发模式，开发的时候碰到了一些痛点

   1. Android各个版本的WebView差异太大，4.4以下甚至使用传统的WebKit而不是Chromium内核，前端需要适配各种机器，各种兼容性的问题让人抓狂。
   2. 对内核有定制需求根本不可能。App开发只能靠大量的javascript注入实现一些简单的功能，有很多非常需要的需求没法落地。
   3. 超级App对自主内核有强烈的需求，如微信
   
   一般的App公司，要基于Chromium做出一个自主的内核来，难度很大。特别是Android5.0版本在Framework层对Render做了线程化，对显示的架构影响比较大。而Android WebView
   的机制，网页绘制又是通过在Framework的Render里面置入DisplayList（相当于一些列的OpenGL command)来实现的。Chromium M39后的的内核，硬件加速这块很难在5.0以前的
   版本上跑起来。而5.0以下的版本，硬件加速是否开启，对网页绘制性能的影响很大。所以，要通过自己直接一直Chromium内核和Android WebView的方式来解决自主内核的问题，
   要投入很大的人力。
   
   另外，android版本兼容，对自主内核来说，也是一个蛮大的挑战。相比之下，CrossWalk因为其广泛的测试和应用，兼容性势必要好很多。

# 如何使用XWalk Embed

  
# 使用XWalkView的一些注意事项
  1. 缺省情况下，XWalkView的backend是SurfaceView，这决定了XWalkView缺省情况下不能使用动画，同一个Window不能有两个XWalkView
  2. XWalkView需要显示地调用destory等操作
  3. Android6.0以上版本，同一个app，XWalkView和Android WebView不能共存   
