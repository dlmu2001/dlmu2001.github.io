---
layout: post
title: 当独立内核碰到阿里百川 
categories: [开发故事]
tags: [Chromium]
description: 开发故事，独立chromium内核，如何解决第三方SDK接
             口写死android webview，以及第三方接口使用webkit.CookieManager
---

  &emsp;&emsp;通过使用crosswalk，实现了app自主chromium内核，脱离了android webview的限制，能做的事情大大增加，调试也更加方便，对于hybrid app来说，
  
  一切都非常完美。当然，美好的世界，有时候也会来点小插曲，世界才会更加多彩。

  &emsp;&emsp;最近碰到了阿里百川的问题就是这样一个小插曲。

# 问题的起源

  百川电商SDK里面有如下接口:

    /**
    * 打开电商组件,支持使用外部webview
    *
    * @param activity             必填
    * @param webView              外部 webView
    * @param webViewClient        webview的webViewClient
    * @param webChromeClient      webview的webChromeClient
    * @param tradePage            页面类型,必填，不可为null，详情见下面tradePage类型介绍
    * @param showParams           show参数
    * @param taokeParams          淘客参数
    * @param trackParam           yhhpass参数
    * @param tradeProcessCallback 交易流程的回调，必填，不允许为null；
    * @return 0标识跳转到手淘打开了,1标识用h5打开,-1标识出错
    */   
    AliTradeService.show(Activity activity,WebView webView, WebViewClient webViewClient,
                        WebChromeClient webChromeClient, AlicBasePage tradePage, 
                        AlibcShowParams showParams, AlibcTaokeParams taokeParams, 
                        Map trackParam, AlibcTradeCallback tradeProcessCallback); 
   
    
  &emsp;&emsp;为了使用利用手淘的sso登录，我们的app引入了这个接口。
 
  &emsp;&emsp;从一个接口的设计角度看，这个接口的设计实在是太糟糕了，一大堆的输入，Activity

  和WebView都是非常大的结构体，传入这种参数，接口的耦合性必然很大。出了问题，API的使用者调

  试比较困难。API里面权利太大，可以任意设置该WebView，获取WebView/Activity相关的信息。

  &emsp;&emsp;对于独立内核来说，上面的内核还不算严重，更大的问题是，接口绑定了WebView，

  WebViewClient和WebChromeClient。更有甚者，SDK内部调用了webkit.CookieManager来getCookie和

  setCookie。

  &emsp;&emsp;假设一个App使用独立的chromium内核，那么他可能是foo.WebView来作为页面加载，它

  的cookie和android WebView的cookie也是各自独立存储的。这让独立内核情何以堪?

  &emsp;&emsp;App也可以对这个百川的页面单独处理，仍然使用WebView。但我们显然想找到更好的解

  决方案。
  
# 对百川的分析

  1. 百川会重新new了一个WebViewClient/WebChromeClient，然后把自己的client设置到我们传入的WebView中,这两个client和我们传入的client处理的优先级尚不明确。比如在它的WebViewClient::shouldOverrideUrl中，它判断到login.taobao.com就会调用我们的shouldOverrideUrl函数，如果我们返回false，它会进行劫持，转到调用手淘登录。

  2. 登录的过程中，百川会先调用CookieManager.getCookie来获取app的WebView的Cookie，得到一些参数，传给手淘，手淘授权成功以后，再把一些参数通过setCookie　inject到webkit.CookieManager里面。 

  3. 授权成功后，百川SDK会调用WebView::reload重新reload这个页面，这样就可以把授权登录得到的cookie传给服务器。

  4. 在百川内部，会通过我们传入的webview的getSettings做一系列的设置，比如设置user-agent。设置是否接收第三方Cookie等。

# 解决思路

  1. Fake出来一个WebView，用来传给百川。这个WebView不是用来load网页，只是用来和百川对接。

  2. 对这个WebView的所有调用,delegate到我们的XWalkView上面来。方法是重载这个WebView，重写绝大部分public方法。

  3. 对于这个Fake WebView的setWebClient/setWebChromeClient重载，保证百川调用setWebViewClient/setWebChromeClient的时候，可以设置到XWalkView来。从而可以保证XWalkView的回调可以到达百川。

  4. 定制crosswalk的chromium内核，在底层将cookie管理委托给webkit.CookieManager。这样可以保证双方的cookie是同步的。


