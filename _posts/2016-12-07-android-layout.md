---
layout: post
title: Android上的布局选择 
categories: [Android]
tags: [布局]
description: Android常见的布局方式的使用
---

tomorrow.cyz@gmail.com

Android Layout有LinearLayout,RelativeLayout,FrameLayout,AbsoluteLayout,TableLayout等。
本文主要针对最常用的LinearLayout,RelativeLayout,FrameLayout三种。
说起来很简单，但是还是很多人没有认真用好。

# LinearLayout
  * LinearLayout是最简单的布局方式，他把所有的子控件按照某一个方向进行排列。要嘛就是
    行排列(horizontal)，要嘛就是列排列(vertical)

  * 主要属性
    - orientation 属性，默认是horizontal，如果要竖排，就要设置成vertical
    - weight 子控件占的空间比重，性能不好，不建议使用
    - gravity 控件自身内部的位置设置，也就是控件的显示内容在控件内是居中，靠上等。
    - layout_gravity 控件相对于父控件的位置，可以在父控件内居中(center,center_veritical)，靠左(left)等

  * 如果LinearLayout里面有嵌套的LinearLayout，很可能需要考虑下，是否要用RelativeLayout
    来代替

# RelativeLayout
  * 顾名思义，RelativeLayout是根据相对位置进行layout的布局方式，这个相对位置既可以相对于
    相邻元素，也可以相对于父元素

  * RelativeLayout是一种非常强大的布局方式，通过合理的安排，可以减少布局层级，提高性能。
    应该是复杂布局里面最常用的一种布局。没有之一。RelativeLayout.LayoutParams值得认真研究。

  * 比如仅需结合layout_alignParentTop layout_below layout_toLeftOf等，可以布局出各种行列结合
    的复杂布局。
    
>>![RelativeLayout布局](https://developer.android.com/images/ui/sample-relativelayout.png  "RelativeLayout布局")

  * 主要属性
    - 相对于父元素
        * layout_alignParentTop
        * layout_alignParentBottom
        * layout_alignParentLeft
        * layout_alignParentRight
        * layout_centerVertical (centers this child vertically within its parent)
        * layout_centerInParent
        * layout_centerHorizontal
    - 相对于相邻元素
        * layout_below
        * layout_above
        * layout_alignBottom(and layout_alignEnd layout_alignLeft layout_alignRight layout_alignTop)
        * layout_toRightOf(layout_toLeftOf)
        * layout_alignWithParentIfMissing（如果设置的layout_toRightOf/layout_toLeftOf相邻元素不存在，
            就对齐父元素）

# FrameLayout
  * FrameLayout可以理解成帧叠加的布局，也就是在一块区域上，一个叠上一个，上面的元素盖住下面的元素，比如浏览器
      页面显示区域上出现一个放大镜工具这样的情况，或者透明图层。

  * 很多人利用FrameLayout来控制不同的情况下使用不同的层，有点过渡使用了。

  * 主要属性
    - gravity 控件自身内部的位置设置，也就是控件的显示内容在控件内是居中，靠上等。
    - layout_gravity 控件相对于父控件的位置，可以在父控件内居中(center,center_veritical)，靠左(left)等


# 其它常用属性
  * id
  * layout_width 
  * layout_height
  * layout_marginBottom/layout_marginTop/layout_marginRight/layout_marginLeft
  * layout_paddingLeft/layout_paddingRight/layout_paddingTop/layout_paddingBottom
  * padding是设置自身的内容相对于自己的（相当于自己控件是一个box），margin相当于设置控件本身（包括内容和padding）
      相对于父元素(相当于父元素是一个box)



 
