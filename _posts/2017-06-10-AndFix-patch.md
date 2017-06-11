---
layout: post
title: Alibaba AndFix 热更新方案兼容性问题的补丁 
categories: [Android]
tags: [热更新]
description: 阿里巴巴的AndFix热更新方案因为其立即生效的特性非常出众，但是实际中有很多兼容性的问题，阿里巴巴新的解决方案Sophix以一个非常优雅的方式解决了这个问题，可是Sophix需要绑定百川后台。有不少用户只想解决兼容性问题，并不想整体引入Sophix，所以根据阿里巴巴的文章做了一个AndFix5.0的patch，解决兼容性的问题。
---

tomorrow.cyz@gmail.com

在热更新方案里面，阿里巴巴的AndFix因为"立即生效"这样的特性显得非常的出众，但是兼容性一直是它的硬伤，看到[AndFix的issue列表](https://github.com/alibaba/AndFix/issues),就让很多公司望而却步。

最近阿里巴巴以一个非常优雅的方式解决了兼容性这个问题，这个方案的竞争力大增。然而阿里巴巴并没有在开源的AndFix中fix这个兼容性的问题，而是另起炉灶，弄了一个Sophix，而且
绑定百川后台，并不开源。

手机淘宝技术团队MTT的博文[Android热修复升级探索](http://mp.weixin.qq.com/s/Uv0BS67-wgvCor6Fss6ChQ)非常详细地描述了这个解决方案，非常优雅，而且简单。

地瓜根据这篇文章在AndFix5.0的基础上fix了兼容性问题。可以看到，现在只要非常非常少的NDK代码。具体的代码的fix在[dlmu2001/AndFix](https://github.com/dlmu2001/AndFix)这里。

aar在https://github.com/dlmu2001/AndFix/tree/master/outputs这里，用这个aar替换alibaba AndFix的aar就可以。




 
