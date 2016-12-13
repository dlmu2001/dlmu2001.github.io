---
layout: post
title: Android布局优化 
categories: [Android]
tags: [优化]
description: Android布局优化的一些思路 
---

tomorrow.cyz@gmail.com

  &emsp;&emsp;本文仅关注Android 布局优化，不讨论布局之外的优化。

  &emsp;&emsp;布局优化是Android应用优化的一个重要方面，它可以减少应用在UI布局上layout,measure和onDraw的时间，从而减少渲染

  时间。布局优化的原则非常简单，就是减少嵌套层级，尽量让布局扁平化。涉及到的知识也不多，基本上就是官方的[Improving Layout Performance](https://developer.android.com/training/improving-layouts/index.html)

很简单的几篇文章就够了。

  &emsp;&emsp;更多的功底体现在理解你的需求，更好地组织功能，布局和代码上，以及，抠细节。

# 从Android Lint开始

  &emsp;&emsp;Android Lint是个静态代码分析工具，无需运行，即可对项目进行扫描，分析潜在的问题。

  应该把它当做App性能优化的第一步，以及日常代码规范的一部分。它的设置见 

  https://developer.android.com/training/improving-layouts/optimizing-layout.html#Lint

  使用很简单，执行Android Studio的Analysis-->Inspect Code即可,执行完，inspection窗口就

  可以显示潜在问题项。
  
  ![](/assets/media/lint.png) 
  
# 使用Hierarchy Viewer分析布局

  &emsp;&emsp;Hierarchy Viewer可以帮助开发者debug和优化布局，它以可视化的形式展现布局结构和属性，可以结合debug帮助观察特定UI

  对象进行invalidate和requestLayout的过程，还可以导出displayList。

  &emsp;&emsp;Hierarchy Viewer已经集成到Android Studio 中，点选Android Device Monitor可以打开DDMS，在右上角靠近DDMS图标的地方，

  有个Hierarchy Viewer图标，点击可以打开Hierarchy Viewer。如果找不到这个图标，可以reset layout。

  &emsp;&emsp;除了在Android Studio中使用Hierarchy Viewer，还可以使用standalone的Hierarchy View，在Android/Sdk/tools目录下，直接

  运行。Standalone的版本有像素视图，可以观察细节。

  &emsp;&emsp;Hierarchy Viewer右上角的三个球图标可以获取节点layout的时间，点击后，当前选中的节点以及它的子节点都会显示出Measure,

  Layout,onDraw时间，在节点上显示三个球，分别对应这三个时间。红色和黄色代表相对性能较差的节点，相对性能较差不不一定

  有问题，但值得你花更多时间关注。

  ![](/assets/media/hierarchyviewer.png)

# 使用include标签重用布局

  &emsp;&emsp;使用include可以重用你的UI部分（相当于把他们组件化），比如，App有N个界面，每个界面都需要一个TitleBar，可以把TitleBar

  独立出来，在各个页面中include。

  &emsp;&emsp;这样做的好处是可以减少维护成本，只要改一个地方，就全局生效了。另外，如果你想要一个其它风格的TitleBar，在主布局修改
 
  include的layout src就可以了。

  &emsp;&emsp;如果你所有的界面TitleBar还有细微的差别，将共性的东西抽出来，在里面留出可以定制的部分，让每个界面去客制化。

  当然，重用也有度，如果差别太大，重用会引入一些太多不需要的组件，就不要重用。
  
  &emsp;&emsp;include的使用很简单，直接使用include标签，设置layout属性就可以。如下
 
  <include layout="@layout/titlebar"/>
 
  通常在titlebar这个布局文件里面设置layout_height,layout_width等属性，也可以在include标签里面指定覆盖它。

    <include layout="@layout/titlebar" 
                   android:layout_width="match_parent"
                   android:layout_height="40dp"
                   android:id="@+id/main_title"/>

  
# 使用merge标签
   
   &emsp;&emsp;很多开发人员使用include标签的时候，上来就先LinearLayout或者RelativeLayout，作为容器。然后在容器里面再加控件。

   其实在很多情况下，这个作为容器的标签是可以优化掉的。merge标签就是用来在此时帮助消除重复的container的。

   比如
    
        <merge xmlns:android="http://schemas.android.com/apk/res/android">
            <Button ..../>
            <Button ..../>
        </merge>

   直接使用include的parent节点作为Button的container。

# 使用ViewStub实现需要时加载

  &emsp;&emsp;有时候，你UI中的某些元素，你只有在某些情况下才会展现。比如某些设置的高级功能，而你又想将所以的设置放在一个页面。
   
  这个时候可以考虑放置一个”高级“或者”更多“标签，点击一下，把剩下的选项show出来。大多时间，你不会展示这些高级选项。

  很多开发者第一时间想到setVisibility来实现。

  &emsp;&emsp;相比之下，ViewStub是更好的选择，setVisibility(GONE)虽然不参与排版，但是控件会申请内存，要初始化，findViewById的
   
  遍历也要加深，这些都是代价。ViewStub就像它的名字一样，仅在那个地方打个桩，不仅不参与绘制，UI组件也不会初始化。在你需要展示

  的时候，才会初始化控件，分配内存。

  &emsp;&emsp;ViewStub有个限制，就是不支持在它的layout里面使用merge标签。

  ViewStub的使用如下:

  &emsp;&emsp;布局里面声明

            <ViewStub
                android:id="@+id/stub_import"
                android:inflatedId="@+id/panel_import"
                android:layout="@layout/progress_overlay"
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_gravity="bottom" />
       
  &emsp;&emsp;代码里面加载

        ((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
        // or
        View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();

  &emsp;&emsp;相应的，在加载的时候再去获取子控件

        Button btn = (Button)importPanel.findViewById(R.id.btn);
            

# 一些值得考虑的地方
  * 很多开发人员喜欢通过引入一个组件改变多个控件的属性，这种情况的确存在，比如在一个横排的LinearLayout中突然插入一个

    竖排的LinearLayout，但是更多的情况，是可以通过巧妙设计控件的属性来达到。所以，碰到这种情况，值得仔细推敲下。甚至

    在这个例子中，也有很多情况下可以通过RelativeLayout来实现。

  * 布局的原则是尽量扁平，减少嵌套
  
  * 减少LinearLayout layout_height参数的使用，Android官方已经明确这个属性会降低layout的效率。
  
  * 如果只是一次性弹出来的部分，比如一个提示界面，适合用动态的方式实现，比如用PopWindow，或者addView/removeView，而不

    要当成布局的一部分，用setVisibility来实现。

  * 有些应用在不同场合下会显示不同的btn，这时候可以考虑只用一个ImageButton，动态设置ImageButton的图片（setImageResource）。

  * 对LinearLayout的嵌套思考再思考
  
  * 使用compound drawables(可以理解为图文按钮)，比如 

        更多  >>
 
       可以通过setDrawableRight来实现，一个组件就可以，而不是如下三个组件
       
        LinearLayout

            TextView
        
            ImageView
         
      这一点Android Lint会有提示
  
  * 建议深刻理解RelativeLayout布局

参考

1. http://blog.csdn.net/ljz2009y/article/details/22690935

2. http://blog.csdn.net/ddna/article/details/5527072

3. https://developer.android.com/training/improving-layouts/optimizing-layout.html

