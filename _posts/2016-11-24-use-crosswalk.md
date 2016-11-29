---
layout: post
title: Android上使用Crosswalk，实现app内核定制 
categories: [Android]
tags: [Chromium]
fullview: true
---

tomorrow.cyz@gmail.com

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
  [XWalkView](https://crosswalk-project.org/apis/embeddingapidocs/reference/org/xwalk/core/XWalkView.html)有一个例子很好地展示了XWalk Embed的使用。
  
   Crosswalk的API和Android WebView并没有一一对应的关系，官方也明确声明，出于兼容性和性能的考虑，并不完全兼容WebView的API。但是，大部分Android WebView的接口，在XWalk都能找到。比如，XWalkView类对应于WebView类，XWalkSettings对应于WebSettings,XWalkNavigationHistory对应于WebBackHistory，XWalkCookieManager对应于CookieManager。CookieSyncManager本身在Android API21已经Deprecated，所以没有对应的XWalk类。
  
   对应于WebView的Callback WebViewClient和WebChromeClient，XWalk提供了XWalkResourceClient和XWalkUIClient类。大部分接口都可以对应得上。需要注意的是，XWalkResourceClient里面，onLoadFinished对应的是整个页面级别的Finish，而不是单个Resource的Finish，而onLoadStarted则对应的是单个resource的Finish。对应于单个页面的start的接口是XWalkUIClient::onPageLoadStarted。
  
   XWalk没有对应的onGeolocationPermissionsShowPrompt接口。
  
   官方文档有的地方没有更新（包括源代码的注释也没有更新），说如果不继承XWalkActivity的话，需要自己调用XWalkInitializer来初始化。已经确认，并不需要。XWalk并不需要一个初始化的接口，从源代码实现上可以看到XWalkView初始化的时候，如果内核没有初始化，会进行初始化（比如load library)。正因为如此，第一个XWalkView的调用会略慢一点，但并不明显。
  
   XWalkView和WebView的使用的主要差异有两点：
   
   1. XWalkView需要显示地调用destroy等操作，在这个Class Overview的例子里面可以看到。
   2. XWalkView的初始化需要一个Activity的参数，这限制了XWalkView在一些Null Acitivity的使用场合，比如Service。WebView则没有这方面的限制。

# 自己编译crosswalk内核
  自己编译Crosswalk，给了我们定制化内核的能力，比如裁剪一些功能以减少size，比如实现password save等定制化需求。
  另外，自己编译Crosswalk，碰到问题可以从内核上调试及解决，不怕坑。
  [编译Crosswalk](https://crosswalk-project.org/contribute/building_crosswalk_zh.html#contribute/building_crosswalk/Building-Crosswalk-for-Android) 给出了很detail的步骤。
  
# 使用XWalkView的一些注意事项
  1. 缺省情况下，XWalkView的backend是SurfaceView，这决定了XWalkView缺省情况下不能使用动画，同一个Window不能有两个XWalkView。
     这个问题在App利用Fragment进行页面切换的时候非常容易暴露出来，比如，一个App在某一个操作的时候，通过隐藏一个Fragment，显示另外一个Fragment来加快不同页面的切换速度（相比之下，Activity的切换成本显然要高），如果两个Fragment里面都有XWalkView，第二个Fragment就会显示不出来。
     
     官方对这个问题的解决方案是开发者可以通过设置backend为TextureView，来代替SurfaceView。代价是一定的性能损失和内存的增加。
     
     XWalkPreferences.setValue(XWalkPreferences.ANIMATABLE_XWALK_VIEW, true);

     [xwalk-2457](https://crosswalk-project.org/jira/browse/XWALK-2457)有对这个问题的bug描述

     TextureView和SurfaceView都会使用硬件加速，但是SurfaceView可以在没有硬件加速的情况下工作，TextureView则不能。
 
     也可以参考[ANIMATABLE_XWALK_VIEW](https://crosswalk-project.org/apis/embeddingapidocs/reference/org/xwalk/core/XWalkPreferences.html#ANIMATABLE_XWALK_VIEW)
     和[chromium的讨论](https://groups.google.com/a/chromium.org/forum/#!topic/graphics-dev/Z0yE-PWQXc4)

  2. XWalkView需要显示地调用destroy等操作
    如何使用XWalk Embed里面已经有描述，需要特别注意

  3. Android6.0以上版本，同一个app，XWalkView和Android WebView不能共存
   在Android系统里面，每个App全局只有一个Asset目录，而Chromium内核初始化icu的时候（lazy-init，要调用icu的地方初始化icu），会将icudtl.dat目录拷贝到Asset目录下，两个chromium内核都要拷贝自己的icudtl.dat到asset目录下，就会发生冲突。
   这个问题要考虑到其他第三方包对android webview的使用，比如自己的app可能没有调用任何android.webkit的函数，但是引入了第三方包exampleSDK，该第三方包调用了CookieManager或者CookieSyncManager，就会带来共存问题。直接导致你的app在有些输入框界面crash（因为icu resource missing)。
   官方给的workaround是在android webview之前初始化XWalkView，是一个很差的解决方案。首先，第三方SDK大部分在Application初始化的时候就调用，而XWalkView的初始化要一个Activity参数，导致需要将第三方SDK的初始化放到第一个Activity的onCreate里面运行，然后实例化一个可能没有用的XWalkView。另外，这样的解决方案其实破坏了Android WebView，如果其他第三方SDK在之后内部调用到Android WebView且有同ICU相关页面，那么crash的概率很大。
   更好的解决方案是自己编译内核，将XWalkView的icudtl.dat重命名。
   相应的bug可以参考[xwalk-7004](https://crosswalk-project.org/jira/browse/XWALK-7004)。
   对ICU没有概念的看下[Chromium ICU库定制裁剪](http://blog.gclxry.com/custom-chromium-icu-library/)   

tomorrow.cyz@gmail.com

dlmu2001.github.io

    


