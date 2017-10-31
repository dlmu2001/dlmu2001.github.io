---
layout: post
title:  创业公司Android App架构 
categories: [Mobile]
tags: [架构]
description: 谈谈创业公司的Android App架构 
---

tomorrow.cyz@gmail.com 

# 1.创业公司需要对App进行架构吗？
&emsp;&emsp;创业公司前期盈利模式、运营方向都未定，一般都要求快速迭代，很多会牺牲产品质量来追求速度，
更有甚者，一个产品做个一两个月，上线，运营下，数据不好，马上就停止，换另一个产品进行开发。
这个时候，真的要进行架构吗？

&emsp;&emsp;这个问题仁者见仁，我的看法是创业公司一定要进行App架构，但是架构的时候不要依照大公司的
方式，而要在完成需求的同时完成架构，尽量轻量级，保留一定的灵活性，但是不要为了灵活性浪费太
多时间。一个良好的轻量级架构应该是节省你的开发时间，或者不应该增加太多开发时间。

&emsp;&emsp;App架构的时候，如何抽象业务是很重要的一部分，但是这一部分个体差异较大，这篇文章侧重于业务
无关的通用架构。

# 2.面向产品和运营架构
&emsp;&emsp; 一个好的开发人员应该有产品和运营的意识，所以架构/软件设计的时候，一定要有一根弦，我做出来
的产品最终要靠产品和运营来落地，特别是中国现在大部分互联网/App产品都是靠运营/产品驱动而不是
技术驱动的情况下。在架构取舍的时候，产品/运营角度是优于技术进行考虑的。

&emsp;&emsp;比方说，帐号注册/登录系统可以在很多APP都是第一个要考虑的事情，特别能考研开发是不是足够有经
验，经验不够的开发人员评估，会觉得很简单，评估出来的工作量一般偏低，架构出来的东西会偏简单实现，
这个地方要考虑到方便运营，涉及到比如拉新的注册转化率，电商产品里面支付转化率，运营会有各种手段，
一定要足够灵活。UI层面上能够以wiget实现为佳，注册/登录的各种API/事件要方便插拔。这个可以参考
facebook/google/instam等第三方登录API。

&emsp;&emsp;题外话，一个好的注册登录设计可以提升注册转化率，支付转化率,账户安全性，即使在时间很紧迫的时候，
也值得多花时间

# 3.迭代架构
&emsp;&emsp;架构不可能一步到位，可以把他视为一个产品，他是否适合公司产品/运营的需求也需要不断磨合，不要停止
迭代的脚步，在迭代架构的时候，注意控制它的影响范围，避免步子大了扯到蛋的情况。后端现在很多都用微服务来减少
依赖，降低耦合。App的架构设计的时候，也尽量设计成子系统的形式，各个子系统职能尽量分明，

&emsp;&emsp;输入输出清晰，这样子迭代架构的时候，容易做到无痛替换。

# 4.开发模式
&emsp;&emsp;开发模式根据业务和团队现状来定，纯原生的应用现在越来越少，它的迭代周期、部署便利性和人员投入让它
的模式比较重，但是有的创业公司来web开发都没有，没得选择。而在混合开发中，h5还是rn、weex，哪种模式都有其优缺点，
不要太过于纠缠这个问题，创业的前期，主要是为了探索业务方向，这三种模式应该都能够满足，退一步来说，即使选了一
种模式，并不妨碍团队继续使用另一种模式，所以选择的出发点，应该更多立足于团队的技能 现状，哪个熟悉用哪个。 

# 5.mvp/mvvm
&emsp;&emsp;mvp/mvvm最近几年被很多Android开发人员等同于Android应用架构了，然而真正领会到的并不多，比如很多所谓的
mvp分层，就只有v和p，m就是一个bean，根本不是mvp设计的初衷。在mvp/mvvm这个问题上，我的看法是所有的模式，归根到底
就是解决三个问题，一是解偶，二是团队统一一个代码设计风格，减少沟通成本，三是自动化测试。
  
&emsp;&emsp;在mvp/mvvm具体怎么写上，google有官方推荐写法[Android Architecture Blueprints](https://github.com/googlesamples/android-architecture),
可以照猫画虎，写几个就会明白为什么要这么写了。如果团队是多人合作，大家都按照同样的写法来写，代码结构会非常清晰，团
队成员互相维护别人的代码很容易。

# 6.打点
&emsp;&emsp;打点是很多App的一个必不可少的需求，运营/产品需要根据用户数据来修正方向，需要对用户行为/用户路径进行分析。
最常见的需要统计日活，pv,转化率等。所以App打点这块涌现了一些方案，比如说各种无痕打点/可视化打点。想要解决的主要是希望
点位的维护（打点，去点）不需要程序员干预，不需要发版。对于精细化埋点，需要获取很多具体参数，这些方案并不能很好的解决，
他们解决的更多的是通用打点，比如pv，或者按钮打击事件这样的非精细化埋点，不过即使是这样，

&emsp;&emsp;对运营/产品也有很大的帮助。特别是创业前期，还没有达到精细化运营的阶段，精细化埋点并不多，更多的是研究pv和用户
浏览路径。
 
&emsp;&emsp;在埋点上，我倾向于页面打点，而不是行为打点，比如说，button a点击可以跳转页面f，button b点击也可以跳转页面
f，这样统一在页面f进行打点就好了，跳转的地方带上参数src，标明从哪里跳过来了，这样更方便运营进行分析。

&emsp;&emsp;相对于可视化埋点，有一个更轻量级的解决方案，在baseactivity/basefragment里面，统一调用打点接口，点位名称使用
类全名，点位参数包括bundle里面的所有参数（通常我会和Router结合起来)。同时，在打点接口的时候，构造一个hashmap，
服务器端下发hashmap，比如"com.android.test.mainactiviy:main_page",com.android.test.mainactiviy对应类全名，main_page
对应点位名称，在打点那里则统一处理下，如果hashmap[类名]能够找到对应的点位名称，再去打点，否则不打点。这样基本就
可以解决页面类的动态打点。产品/运营只要配一下页面和点位名称就可以进行打点，不需要发版。
 
&emsp;&emsp;页面之外，如果想对控件也进行动态化打点，可以参考[Android无埋点数据收集SDK关键技术](http://www.jianshu.com/p/b5ffe845fe2d),
实现控件唯一标识。

# 7.动态化与热修复
&emsp;&emsp;动态化和热修复更多地是对于原生代码而言的。

&emsp;&emsp;动态化更多地是侧重于灵活地通过服务器端配置，控制App的显示与逻辑。在前期，为了快速迭代，可能利用json加App端的简单的工厂
模式来实现。对于重运营，页面变化频繁的App，比如电商的App，就显得力不从心。大公司为了这类需求，造了不少轮子，比如天猫的
vLayout(七巧板)，最近腾讯开源的RapidView。

&emsp;&emsp;热修复则更多的是为了解决线上重要的bug，主流上根据是否支持即时生效（是否需要重启）分成两类。支持即时生效的方案，包括阿里
的Sophix和美团robust，不支持的，主要包括腾讯的tinker和nuwa（qq空间）。阿里的Sophix采用了底层指针替换的方案，因为指针替换很难原子操作，
本质上是存在不可预料的风险的。而美团的robust,则相当于入侵每个函数，在函数执行的时候先检查是否要修复，相对稳定，但缺点也非常明显，会把
Java的inline优化干掉，而且会大大增加函数个数。

&emsp;&emsp;相对而言，非即时生效付出的代价没有那么大，所以如非必要，我建议尽量使用非即时生效的方案，tinker或者nuwa的方案，都比较成熟。
   
&emsp;&emsp;热更新我有过比较，可以参考以下[Android 热更新机制研究](Android 热更新机制研究)
   
&emsp;&emsp;需要注意的是，不管采用什么方案，都没有办法达到百分百修复。

# 8.结构样式分离
&emsp;&emsp;结构样式分离是web前端的概念，原生在这一块并没有web那么顺滑，但是如有可能，尽量使用style/theme执行这个思路，特别是在美工确定
了App视觉规范以后，这会让你的App看起来越来越专业，后期页面也会做的越来越快。

# 9.路由
&emsp;&emsp;路由一开始也是Web前端的概念，但是最近两年，原生里面很多引入了这一概念，路由主要有两个好处，一是代码解耦，把一个原生的页面也当成一
个url，是一个字串，就像一个接口，在页面没写好之前就可以约定好接口，并写好跳转，另外一个，方便服务器端或者web端调用，比如说，通过push要打开一
个本地的页面，此时就传这个页面的url，搭配对应的参数（可以放在url的query里面），或者在web端调用原生的页面，都非常方便。

&emsp;&emsp;通常情况下，暴露一个RouteConstant.java文件，服务器或者web端就一目了然。
        
        public class RouteConstant{
              public final static String PAGE_MAIN="mytest://main";
              public final static String PRODUCT_LIST=“mytest://productlist";

              public final static String KEY_ID="id";
              public final static String KEY_TITLE="title";
        }
        
&emsp;&emsp;搭配第三方路由框架[chenyu/Router](https://github.com/chenenyu/Router),可以这么调用
                Route.build("mytest://productlist").with("KEY_ID","12345").go(getActivity());

&emsp;&emsp;push直接发送消息(url,id)就可以直接跳转到对应页面，不需要很多个if/else语句。

&emsp;&emsp;web端直接调用iframe或者console.prompt直接调用"mytest://productlist?id=12345"，统一处理下query参数转成跳转参数就可以。

# 10.其它
  
## 10.1 缺陷统计/上报

## 10.2 log控制

## 10.3 开源推荐

    * 路由框架[chenyu/Router](https://github.com/chenenyu/Router)

    * [YoKeyword/Fragmentation](https://github.com/YoKeyword/Fragmentation),fragment有很多坑，不想花时间，用这个很合适。

    * RxJava+Retrofit,这个地球人都知道了，RxJava+MVP/MVVM，可以降低复杂性，把开发的重心聚焦在业务上。

    * [CymChad/BaseRecyclerViewAdapterHelper](https://github.com/CymChad/BaseRecyclerViewAdapterHelper),RecyclerView adapter的一个
       
       封装，可以提高写list的效率，减少很多代码。

   * [ButterKnife](http://jakewharton.github.io/butterknife/),不要写很多findViewById了，IDE还支持一键生成代码。点击变量可以还可以很快
      
      地定位到xml，用得太普遍了。

   * [Trinea/android-common](https://github.com/Trinea/android-common),一些库函数的再封装，我习惯于先找找看trinea实现了没有，有实现就
   
      用他的，没实现自己再增加。
