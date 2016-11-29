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
   版本上跑起来。而5.0以下的版本，硬件加速是否开启，对网页绘制性能的影响很大。所以，要通过自己直接移植Chromium内核和Android WebView的方式来解决自主内核的问题，
   要投入很大的人力。
   
   另外，android版本兼容，对自主内核来说，也是一个蛮大的挑战。相比之下，CrossWalk因为其广泛的测试和应用，兼容性势必要好很多。

# 如何使用XWalk Embed
    [XWalkView的Class Overview](https://crosswalk-project.org/apis/embeddingapidocs/reference/org/xwalk/core/XWalkView.html)有一个例子非常好地展示了XWalk Embed的使用。
    Crosswalk的API和Android WebView并没有一一对应的关系，官方也明确声明，出于兼容性和性能的考虑，并不完全兼容WebView的API。但是，大部分Android WebView的接口，在XWalk都能找到。比如，XWalkView类对应于WebView类，XWalkSettings对应于WebSettings,XWalkNavigationHistory对应于WebBackHistory，XWalkCookieManager对应于CookieManager。CookieSyncManager本身在Android API21已经Deprecated，所以没有对应的XWalk类。
    对应于WebView的Callback WebViewClient和WebChromeClient，XWalk提供了XWalkResourceClient和XWalkUIClient类。大部分接口都可以对应得上。需要注意的是，XWalkResourceClient里面，onLoadFinished对应的是整个页面级别的Finish，而不是单个Resource的Finish，而onLoadStarted则对应的是单个resource的Finish。对应于单个页面的start的接口是XWalkUIClient::onPageLoadStarted。
    XWalk没有对应的onGeolocationPermissionsShowPrompt接口。
    官方文档有的地方没有更新（包括源代码的注释也没有更新），说如果不继承XWalkActivity的话，需要自己调用XWalkInitializer来初始化。已经确认，并不需要。XWalk并不需要一个初始化的接口，从源代码实现上可以看到XWalkView初始化的时候，如果内核没有初始化，会进行初始化（比如load library)。正因为如此，第一个XWalkView的调用会略慢一点，但并不明显。
    XWalkView和WebView的使用的主要差异有两点：
    1. XWalkView需要显示地调用destroy等操作，在这个Class Overview的例子里面可以看到。
    2. XWalkView的初始化需要一个Activity的参数，这限制了XWalkView在一些Null Acitivity的使用场合，比如Service。WebView则没有这方面的限制。
    {% highlight java %}
    {
        XWalkView walkView=new XWalkView 
    }
    {% endhighlight %}
# 自己编译crosswalk内核
  自己编译Crosswalk，给了我们定制化内核的能力，比如裁剪一些功能以减少size，比如实现password save等定制化需求。
  另外，自己编译Crosswalk，碰到问题可以从内核上调试及解决，不怕坑。
  [编译Crosswalk](https://crosswalk-project.org/contribute/building_crosswalk_zh.html#contribute/building_crosswalk/Building-Crosswalk-for-Android)给出了很detail的步骤。
  
# 使用XWalkView的一些注意事项
  1. 缺省情况下，XWalkView的backend是SurfaceView，这决定了XWalkView缺省情况下不能使用动画，同一个Window不能有两个XWalkView。
  2. XWalkView需要显示地调用destroy等操作
  3. Android6.0以上版本，同一个app，XWalkView和Android WebView不能共存   
