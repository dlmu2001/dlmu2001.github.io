---
layout: post
title: Android系统上两个chromium内核共存的问题 
categories: [开发故事]
tags: [Chromium]
description: 开发故事，android系统上两个chromium内核共存问题的发现及解决
---

tomorrow.cyz@gmail.com

  &emsp;&emsp;[Android上使用crosswalk，实现App内核定制](https://dlmu2001.github.io/android/2016/11/24/use-crosswalk.html)一文中，
  我们在注意事项里面提到了Android6.0以上版本中，XWalkView和Android WebView不能共存在Android系统里面的问题。这个问题的发现和解决
  颇有意思。

# 问题的发现及debug
 
  &emsp;&emsp;在集成crosswalk以后，测试中发现，对于Android6.0以上的机型，某些页面显示乱码。

  &emsp;&emsp;碰到乱码问题，第一时间想到的是charset。看了下页面，gbk的页面有乱码现象。

  &emsp;&emsp;考虑到crosswalk是一个成熟的内核，不可能对gbk编码的支持都存在问题。因此怀疑是我的app的问题。为了验证这个判断，基于

  crosswalk写了一个最简单的demo，去load这个gbk编码的页面，果然，不会出现乱码。

  &emsp;&emsp;基于这个判断，我开始找app中可能和编码有关的地方，然而并没有什么发现。于是，还是得从编码开始调，为了简化步骤，想到了
 
  两个方法，第一个是load最简单的gbk编码的本地网页，包含一个或那个字，跟踪编解码的过程。另一个就是从input text入手，因为input text

  可以自己输入字串，另外也同样涉及到编解码的处理。结果在第二条路径有意外发现，在input text的时候，系统居然crash了。

  &emsp;&emsp;分析了下breakpad，堆栈已经乱掉，只有最后的一级调用，是一个非常通用的函数,blink::findWordBoundary。这个函数调用的地方

  非常多。但是看起来应该是和文本处理有关，极有可能和编解码也有关系。

  &emsp;&emsp;这个crash非常珍贵，它可以把我从枯燥的编解码调试中解放出来。为了获得完整的调用堆栈，我挂上gdb。得到了调用堆栈。

    #0 blink::findWordBoundary (chars=chars@entry=0x95f7de36, len=len@entry=1, position=0, start=start@entry=0x95f7ccd0, end=end@entry=0x95f7ccd4)
    at ../../third_party/WebKit/Source/platform/text/TextBoundaries.cpp:95
    #1 0x9bc87288 in blink::startWordBoundary (characters=0x95f7de36, length=1, offset=<optimized out>, mayHaveMoreContext=blink::MayHaveMoreContext, needMoreContext=@0x95f7cd33: false)
    at ../../third_party/WebKit/Source/core/editing/VisibleUnits.cpp:860
    #2 0x9bc89b74 in blink::previousBoundary<blink::EditingAlgorithm<blink::NodeTraversal> > (c=..., 
    searchFunction=0x9bc871e1 <blink::startWordBoundary(UChar const*, unsigned int, unsigned int, blink::BoundarySearchContextAvailability, bool&)>)
    at ../../third_party/WebKit/Source/core/editing/VisibleUnits.cpp:706
    #3 0x9bc89cf2 in startOfWordAlgorithm<blink::EditingAlgorithm<blink::NodeTraversal> > (side=blink::LeftWordIfOnBoundary, c=...) at ../../third_party/WebKit/Source/core/editing/VisibleUnits.cpp:879
    #4 blink::startOfWord (c=..., side=side@entry=blink::LeftWordIfOnBoundary) at ../../third_party/WebKit/Source/core/editing/VisibleUnits.cpp:884
    #5 0x9bcb0aec in blink::SpellChecker::updateMarkersForWordsAffectedByEditing (this=this@entry=0x479c19e0, doNotRemoveIfSelectionAtWordBoundary=<optimized out>)
    at ../../third_party/WebKit/Source/core/editing/spellcheck/SpellChecker.cpp:690
    #6 0x9bc798e0 in blink::Editor::insertTextWithoutSendingTextEvent (this=<optimized out>, text=..., selectInsertedText=<optimized out>, triggeringEvent=0x439cbb78)
    at ../../third_party/WebKit/Source/core/editing/Editor.cpp:801
    #7 0x9bc7a8de in blink::Editor::handleTextEvent (this=0x439c1a98, event=event@entry=0x439cbb78) at ../../third_party/WebKit/Source/core/editing/Editor.cpp:182
    #8 0x9baeb3a0 in blink::EventHandler::defaultTextInputEventHandler (this=<optimized out>, event=0x439cbb78) at ../../third_party/WebKit/Source/core/input/EventHandler.cpp:3467
    #9 0x9bb62d10 in blink::HTMLInputElement::defaultEventHandler (this=0x55263188, evt=0x439cbb78) at ../../third_party/WebKit/Source/core/html/HTMLInputElement.cpp:1253
    #10 0x9bade88c in blink::EventDispatcher::dispatchEventPostProcess (this=this@entry=0x95f7e13c, preDispatchEventHandlerResult=preDispatchEventHandlerResult@entry=0x0)
    at ../../third_party/WebKit/Source/core/events/EventDispatcher.cpp:229
    #11 0x9baded76 in blink::EventDispatcher::dispatch (this=0x95f7e13c) at ../../third_party/WebKit/Source/core/events/EventDispatcher.cpp:131
    #12 0x9bade798 in blink::EventDispatcher::dispatchEvent (node=..., mediator=0x479d8f28) at ../../third_party/WebKit/Source/core/events/EventDispatcher.cpp:51
    #13 0x9baeb382 in blink::EventHandler::handleTextInputEvent (this=<optimized out>, text=..., underlyingEvent=underlyingEvent@entry=0x439cbb18, inputType=inputType@entry=blink::TextEventInputKeyboard)
    at ../../third_party/WebKit/Source/core/input/EventHandler.cpp:3461
    #14 0x9bc79866 in blink::Editor::insertText (this=this@entry=0x439c1a98, text=..., triggeringEvent=triggeringEvent@entry=0x439cbb18) at ../../third_party/WebKit/Source/core/editing/Editor.cpp:789
    #15 0x9bc7b3c8 in blink::Editor::handleEditingKeyboardEvent (this=this@entry=0x439c1a98, evt=evt@entry=0x439cbb18) at ../../third_party/WebKit/Source/core/editing/EditorKeyBindings.cpp:68
    #16 0x9bc7b3f8 in blink::Editor::handleKeyboardEvent (this=0x439c1a98, evt=evt@entry=0x439cbb18) at ../../third_party/WebKit/Source/core/editing/EditorKeyBindings.cpp:74
    #17 0x9baec44a in blink::EventHandler::defaultKeyboardEventHandler (this=0x3ee01ca0, event=0x439cbb18) at ../../third_party/WebKit/Source/core/input/EventHandler.cpp:3264
    #18 0x9bb62d5c in blink::HTMLInputElement::defaultEventHandler (this=0x55263188, evt=0x439cbb18) at ../../third_party/WebKit/Source/core/html/HTMLInputElement.cpp:1194
    #19 0x9bade88c in blink::EventDispatcher::dispatchEventPostProcess (this=this@entry=0x95f7e314, preDispatchEventHandlerResult=preDispatchEventHandlerResult@entry=0x0)
    at ../../third_party/WebKit/Source/core/events/EventDispatcher.cpp:229
    #20 0x9baded76 in blink::EventDispatcher::dispatch (this=0x95f7e314) at ../../third_party/WebKit/Source/core/events/EventDispatcher.cpp:131
    #21 0x9bade798 in blink::EventDispatcher::dispatchEvent (node=..., mediator=0x479d8f08) at ../../third_party/WebKit/Source/core/events/EventDispatcher.cpp:51
    #22 0x9baec612 in blink::EventHandler::keyEvent (this=this@entry=0x3ee01ca0, initialKeyEvent=...) at ../../third_party/WebKit/Source/core/input/EventHandler.cpp:3182
    #23 0x9b89a588 in blink::WebViewImpl::handleCharEvent (this=0x5ce74000, event=...) at ../../third_party/WebKit/Source/web/WebViewImpl.cpp:1188
    #24 0x9b89cb80 in blink::WebViewImpl::handleInputEvent (this=0x5ce74000, inputEvent=...) at ../../third_party/WebKit/Source/web/WebViewImpl.cpp:2228
    #25 0x9c21c480 in content::RenderWidgetInputHandler::HandleInputEvent (this=0x9323df50, input_event=..., latency_info=..., dispatch_type=<optimized out>)
    at ../../content/renderer/input/render_widget_input_handler.cc:323
    ......

    void findWordBoundary(const UChar* chars, int len, int position, int* start, int* end)
    {
        TextBreakIterator* it = wordBreakIterator(chars, len);
        *end = it->following(position);
        if (*end < 0)
            *end = it->last();
        *start = it->previous();
    }

  &emsp;&emsp;findWordBoundary函数中，wordBreakIterator返回了空，所以导致对空指针操作了。而问题是wordBreakIterator输入参数chars,len都是正常的，不应该返回空。

  &emsp;&emsp;wordBreakIterator的实现在TextBreakIteratorICU::wordBreakIterator。看到这个类，自然想到了ICU库的问题。因为之前做chromium移植，有考虑过ICU库

  会不会出现单例的情况，导致两个chromium内核互相影响。再结合到简单的crosswalk demo不出现问题。我开始怀疑是不是ICU什么资源在系统的android 

  webview运行的chromium内核和crosswalk运行的crosswalk内核之间有竞争关系。

  &emsp;&emsp;继续对TextBreakIteratorICU::wordBreakIterator进行debug，发现确实是在load ICU数据文件的时候失败了。会不会是打包的时候没打进去，或者压缩解压

  缩的问题呢?检查了下apk，打包是ok的。而且简单的crosswalk demo没有问题，也排除了压缩解压缩的问题。

  &emsp;&emsp;停下debug，分别google了"crosswalk icudata" "crosswalk webview coexistence"等,发现了[XWALK-7004](https://crosswalk-project.org/jira/browse/XWALK-7004),

  看起来是一个已知bug，根据bug的描述，在简单的demo中重现了这个问题。问题的逻辑确实是两者的共存问题，只要系统中同时有android webview和

  xwalkview，xwalkview的实例化如果发生在android webview的调用之后，就会产生问题。

  &emsp;&emsp;官方给出了一个work around，就是让android webview的调用发生在第一个xwalkview的实例化之后。在App尝试了下，把android webview的调用都干掉了，问题仍然存在。

  &emsp;&emsp;怀疑是第三方SDK先调用了android webview。编译个ROM，把有可能调用的地方，比如WebSettings的getUserAgents，CookieManager,CookieSyncManager都打印了调用堆

  栈。运行下，果然，多个第三方SDK初始化的时候调用到了webkit.CookieManager的接口，而这些SDK的初始化都是在Application初始化的时候执行的。

  &emsp;&emsp;改了下这些SDK初始化的位置，问题确实得到了解决。

# 问题的解决
  
  &emsp;&emsp;虽然这个workaround使问题得到了解决，但是这样的处理让app的灵活性大打折扣，另外，把XWalkView放到前面会不会导致如果后面又第三方用到
  
  android webview出现问题呢？毕竟国内很多第三方SDK经常使用WebView，比如阿里百川，比如友盟分享库。催了下crosswalk，这个bug的进展很慢。决定自

  己动手，丰衣足食。

  &emsp;&emsp;开始分析chromium内核中的ICU处理模块，另外也在chromium-dev邮件列表咨询这个问题。有人提到了chromium的icu数据文件load到了android asset

  目录，而对于app来说，这个asset目录是全局的，也就是说chromium的icu数据文件和crosswalk的数据文件如果都load到asset目录，就存在互相覆盖的问题。

  这个分析非常靠谱。

  &emsp;&emsp;于是结合代码，找到并解决了问题，并提交了commit到crosswalk。  


