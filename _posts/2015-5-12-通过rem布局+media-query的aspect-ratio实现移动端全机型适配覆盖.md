---
layout: post
title: 通过rem布局+media-query:aspect-ratio实现移动端全机型适配覆盖
tags: CSS,Html
keywords: css,media-query,rem,responsive,aspect-ratio,mobile,移动端,适配
date: 2015-05-12 20:20
---

在我前一篇关于移动端适配方面的文章里介绍过rem布局，（rem这玩意不造什么开始的，现在貌似现在很火...）

不熟悉的同学可以先看看：[关于移动端自适应页面的一些总结]({% post_url 2014-08-09-关于移动端自适应页面的一些总结 %})。

在文章最后的补充里，我提到的rem配合media query的aspect-ratio使用，来达到全分辨率适配覆盖的目的，在这里就展开来详细介绍一下这种用法。

<!--more-->

----------
### 正文开始
----------

我们知道，当使用rem布局，并配合JS动态按照比例设置HTML标签上的rem值时，整个页面是按照屏幕的宽度来整体缩放的。

对于高度不限制的页面（也就是超出一屏页面可以滚动），这种方案没有任何问题，简单暴力。

但是对于一些需要布满一整屏的页面（比如现在流行的上下/左右滑动的那种），由于一个页面的元素必须在一个页面内布满，不能超出，那么问题就来了：

如果你是按照iPhone5的尺寸来做的页面，尺寸是320*568，那么当页面放在宽度较宽，或高度较矮的手机里（宽高比变大），必定会出现页面元素在一屏放不下的情况（对于上下绝对定位，中间拉伸的页面，就会是挤做一堆）。

可以看下面这个例子，这个页面上面部分是从上往下布局，下面按钮采用bottom绝对定位。

页面是按照标准iphone5尺寸开发的，但是在带有导航条的iphone4下，高度只有411px，由于iphone4/5都是320px的宽，就会发生页面挤作一堆的情况。

<img src="{{ site.url }}/downloads/images/rem/iphone5.jpg" style="width:200px;vertical-align:top;" alt="标准iphone5下">
<img src="{{ site.url }}/downloads/images/rem/iphone4.jpg" style="width:200px;vertical-align:top;" alt="带浏览器导航条的iphone4下">

图1 同一页面在标准iphone5下与带导航条的iphone4下

那么怎么办呢？首先我想到的当时是media query。最开始我想当然的使用max-height/min-height来控制样式，但很快我发现我太单纯了...

因为页面是整体根据宽度缩放的，不同的宽度下的设备，对应的就是不同的高度比例！在我写了若干套max-height/min-height，还会冒出各种各样奇葩分辨率后，我发现这不是办法。

既然我们的页面是根据宽度进行等比缩放的，那么如果能判断宽高比就好了，一顿查阅文档，让我找到了aspect-ratio这个属性，正是我要的！

下面是MDN上[关于aspect-ratio的介绍](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Media_queries):

> #### 宽高比（aspect-ratio）
>
> 值：`<ratio>`
>
> 媒体： `visual`, `tactile`
>
> 是否接受 min/max 前缀：是
> 
> 描述了输出设备目标显示区域的宽高比。该值包含两个以“/”分隔的正整数。代表了水平像素数（第一个值）与垂直像素数（第二个值）的比例。

Ok，有了它以后，事情就变得简单了。

先按照正常iphone5尺寸（320px * 568px）进行页面开发，最后适配时，不需要改变宽度（因为我们适配的是宽高比，而高度是自动根据宽度缩放的），直接调整高度。

对于上面那个例子，我当时是将页面划分为了几个高度区域（单位都是px）：

- 568 ~ 510
- 510 ~ 490
- 490 ~ 470
- 470 - 440
- 440以下

（为什么对568以上的尺寸不做调整？因为我们的页面是根据宽度缩放的，我们只怕高度小，而不怕高度过大）

然后开始写media query。下面以568~510区间为例，看看具体media query : aspect-ratio怎么写。
    
（需要注意的是，aspect-ratio是宽度/高度，而我们关注的是高度，必须要小心高度是分母，对于max-aspect-ratio/min-aspect-ratio不要搞反了，一开始调研的时候我就因为写反了，以为这个属性不好使...）

    @media screen and (max-aspect-ratio: 320/510) and (min-aspect-ratio: 320/568), screen and (-webkit-max-aspect-ratio: 320/510) and (-webkit-min-aspect-ratio: 320/568) {
        // 这里面的样式仅会对320/510 ~ 320/568宽高比之间的设备生效
    }

匹配到样式区间了，那么如何适配？这个嘛，就随意了，只要调整到在这个高度范围内显示正常，那就ok了。

比较简单暴力的，就是直接对主要页面区域施加 `transform: scale(0.8)` 这类样式，直接缩小。

至于怎么划分区间，这个要根据具体的页面布局情况、或者具体你们需要适配的几大机型的宽高比范围来决定，这里并没有一个统一的规定。

### 总结

rem是个好东西，关于怎么使用它最愉快，我也还在探索中。在这里配合aspect-ratio使用，对于简单的需要铺满整屏的移动端页面，是成本最低、最省事、也是最暴力、最彻底的一种方案。

关于rem以及media query: aspect-ratio的兼容性，个人测试是兼容安卓2.3+（加上-webkit-aspect-ratio前缀为了保险起见），并且已经过生产环境上的线上项目校验，兼容性可靠。

关于上面的例子，项目还在线上，有兴趣的同学可以点开看看（Chrome手机模拟或者直接用手机开打哦~）[线上链接](http://map.baidu.com/zt/y2015/baiduPlanet/mobile/)，并且试着改变不同的宽度、高度来查看效果。

有兴趣的同学欢迎尝试~觉得有槽点的也欢迎吐槽~大家一起学习交流。

全文完.