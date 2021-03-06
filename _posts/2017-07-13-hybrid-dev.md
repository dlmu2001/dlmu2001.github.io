---
layout: post
title: hybrid app开发
categories: [Mobile]
tags: [架构]
description: 移动app的hybrid开发模式 
---

tomorrow.cyz@gmail.com

# 1.前言
纯H5的Web App开发模式虽然可以一套代码搞定多个平台，但是在低配机器上性能很差，而且各种兼容性问题对开发人员挑战很大，功能、安全性也受到较大限制。因此，这种开发方式已经不是主流的开发方式。

与之相反，纯Native开发在性能上有优势，用户体验、功能、安全性更好，但是迭代周期和部署是其痛点，而且开发成本高，要针对不同平台开发不同版本。非常不适合目前这种移动开发的节奏。虽然各种组件化，热更新方案层出不穷，但是几乎没有一种能达到Web那样的热部署。

在这种情况下，主流的App都采用hybrid的开发模式，混合Native和Ｈ5。应用中包含了Native代码，用来实现部分H5体验不好，难以实现的逻辑，应用基础框架或者业务复杂、用户使用频繁、体验要求较高的业务（比如首页）。业务相关的，变化频繁的，使用H5进行开发。各家的区别在于Native和H5的占比。

而在hybrid中，为了充分挖掘性能，近几年出现了一些方案，既利用Web的动态部署的机制，又利用原生的渲染机制，从而达到性能上的提升，比如RN。

更早期的时候，为了实现跨平台开发架构，有cordova，phonegap,crosswalk这样的框架。

鉴于学习成本，以及各种框架的局限性，反而是最简单的hybrid开发模式应用最广泛。这种开发模式以WebView/UIWebView/WKWebView为桥梁，将H5和Native联系起来。通过设计通信机制，让H5和Native可以互相调用。

# 2.native调用javascript的方法

## 2.1 Android
* Android在sdk>19以上的版本提供了evaluateJavascript接口
        
        /**
         * Asynchronously evaluates JavaScript in the context of the currently displayed page.
         * If non-null, |resultCallback| will be invoked with any result returned from that
         * execution. This method must be called on the UI thread and the callback will
         * be made on the UI thread.
         *
         * @param script the JavaScript to execute.
         * @param resultCallback A callback to be invoked when the script execution
         *                       completes with the result of the execution (if any).
         *                       May be null if no notificaion of the result is required.
         */
        public void evaluateJavascript(String script, ValueCallback<String> resultCallback); 

就像注释里面说的，这个方法是一个异步执行的方法，必须在UI线程调用，回调会运行在UI线程。回调里面可以
传回json值，json对象或者json数组。[这里](https://github.com/GoogleChrome/chromium-webview-samples.git)
是一个例子。

* Android sdk<=19的版本，可以使用loadUrl的接口, 参数传入javascrip的语句
        
        void loadUrl (String url)

这个接口没有返回值，得自己设计机制，来传回返回值

## 2.2 IOS
* UIWebView的stringByEvaluatingJavaScript方法
         
        func stringByEvaluatingJavaScript(from: String)
这个方法没有返回值
* WKWebView的evaluateJavaScript方法
        
        func evaluateJavaScript(String, completionHandler: ((Any?, Error?) -> Void)? = nil)

这个方法有callback，但是WKWebView目前还有很多限制，不能单纯从是否有返回值来决定使用UIWebView还是WKWebView

# 3.Javascript调用native的方法

## 3.1 Android
* 通过schema的方式，javascript使用iframe发起链接，webview通过shouldOverrideUrl进行拦截
        
        //javascript
        var ifm = document.createElement('iframe');
        ifm.src = 'jsbridge://namespace.method?[...args]';
        document.body.appendChild(iframe);
        setTimeout(function(){
            ifm.remove();
        },100);

        //Android
        public boolean shouldOverrideUrlLoading(WebView view, String url) {
            String scheme = Uri.parse(url).getScheme();//还需要判断host
            if (TextUtils.equals("jsbridge", scheme)) {
                ...
                return true;
            }
            return false;
        }

* JavaScriptInterface
        
        //Android
        class JSInterface {  
            @JavascriptInterface //注意这个代码一定要加上
            public String getUserData() {
                return "UserData";
            }
        }
        webView.addJavascriptInterface(new JSInterface(), "AndroidJS");
        
        //javascript
        alert(AndroidJS.getUserData()) //UserDate 
        
&emsp;&emsp;这种方法在api 17以下存在安全性问题，可以利用javascript代码调用java的反射api，攻击或者操纵设备。
        
        This method can be used to allow JavaScript to control the host application. This is a powerful feature, but also presents a security risk for applications targeted to API level JELLY_BEAN or below, because JavaScript could use reflection to access an injected object's public fields. Use of this method in a WebView containing untrusted content could allow an attacker to manipulate the host application in unintended ways, executing Java code with the permissions of the host application. Use extreme care when using this method in a WebView which could contain untrusted content.
JavaScript interacts with Java object on a private, background thread of this WebView. Care is therefore required to maintain thread safety.
The Java object's fields are not accessible.

&emsp;&emsp;在api 17及以上的版本，可以通过@JavascriptInterface注解解决安全漏洞问题。

* window.prompt

javascript中调用window.prompt(message,value)，用调用的接口通过message参数传递过来。

Android中重载WebChromeClient的onJsPrompt
        
        public class JSBridgeWebChromeClient extends WebChromeClient {
           @Override
           public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
               result.confirm(JSBridge.callJava(view, message));
               return true;
          }
        }

## 3.2 IOS
&emsp;&emsp;通过schema的方式，javascript利用iframe发起链接，UIWebview通过delegate拦截
        
        BOOL webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request 
         navigationType:(UIWebViewNavigationType)navigationType



# 5 hybrid开发的一些常见问题

## 反劫持

## 转菊花

## 多次重定向后退

# 参考
* 1.[QQ空间面向移动时代Hybrid架构设计](http://www.infoq.com/cn/articles/hybrid-app-development-combat)

* 2.[H5与Native交互之JSBridge技术](http://tech.youzan.com/jsbridge/)

* 3.[Android Hybrid App通信](http://www.jianshu.com/p/1cf25c712040)

* 4.[Hybrid APP基础篇(五)->JSBridge实现示例](http://www.cnblogs.com/dailc/p/5931328.html)

* 5.[Hybrid APP架构设计思路](http://segmentfault.com/a/1190000004263182)


