---
layout: post
title:  "关于viewport"
date:   2015-10-26 23:14:00
categories: web
---
#为什么关注这个

看到一些找前端的要求里会写要有移动端开发的经验，做了一年所谓的移动端开发，感觉和PC端的区别并没有太大。而viewport就是其中最先接触到的一点。

正巧看到一篇PPK关于viewport姿势的文章

[传送门part1](http://www.quirksmode.org/mobile/viewports.html)

[传送门part2](http://www.quirksmode.org/mobile/viewports2.html)

[传送门part3](http://www.quirksmode.org/mobile/metaviewport/)

英文的回顾起来比较费力，下面算是笔记

**提前说一句，文中的知识都没什么卵用，赶时间可以直接跳到文章的末尾，那个meta标签应该可以解决你关于viewport的问题**

#正文

##基础
***

###css像素 VS 设备像素

设备像素很好理解，就是数出屏幕上有多少个像素点，可以通过screen.width/height来获取，可以理解为不变的量

css像素就是样式中操作的像素，而当zoom为100%的时候，每一个css像素刚刚好和设备像素一样大。

当你进行拉伸的时候，实际上改变的是每一个css像素的大小，而不是他们的数目，也就是说每一个css像素占用更多个设备像素。

一般来说我们并不关心设备的像素尺寸，因为毕竟我们的页面是按照css像素进行渲染的。

##PC端
***

###viewports

视口可以认为是用来装<html>元素的容器，但视口的大小和html的大小是不等价的。

在PC端，viewport的宽高就是浏览器窗口里面能显示区域的宽高，而一个页面一般比一个屏幕要长，在移动端会更复杂一些

###讨论几个值

`document. documentElement. clientWidth/Height`可以取到viewport的css像素；

`document. documentElement. offsetWidth/Height`可以取到html元素的css像素；

`window.innerWidth/innerHeight`是浏览器窗口的css像素尺寸，与clientWidth/Height不同的是，它包含滚动条；

`window.pageX/YOffset`是页面偏移量的css像素

点击事件带的坐标有`pageX/Y`（相对于html元素计算）,`clientX/Y`（相对于viewport计算）,`screenX/Y`（相对于屏幕计算的设备像素值，没什么卵用）

###media条件判断

适配不同大小屏幕，可能会用到类似`@media all and (max-width: 400px)`之类的条件判断来重写样式。

如果用`width/height`进行条件限制，判断是是视口的尺寸（css像素），如果是用`device-width/device-height`进行条件限制，判断的是屏幕的尺寸（设备像素）。

##移动端
***

###viewports

移动端的问题就是屏幕太小，显示不了太多东西。为此移动端的浏览器有两个视口

>layout viewport

>visual viewport

页面是在layout viewport上面绘制的，尺寸也是在layout viewport上计算，而layout viewport的宽度，是移动端浏览器自己定的，往往比设备屏幕要大得多。

所以我们用户当前看到的部分，就是visual viewport，页面的一部分。通过滑动，缩放来改变visual viewport的偏移量和远近。

*在这些行为中，layout viewport的尺寸没有发生一丁点变化，只是用户通过visual viewport 观察的远近，位置改变了*

想象一下拿着一个放大镜看地图的情景，就知道大概是怎么回事儿了。

多数浏览器在初始化的时候，会将页面缩放到visual viewport与layout viewport宽度相等的状态，也就是缩小到整个页面塞进屏幕里的样子。

两个viewport已经够多了，然而retina屏又让我们需要一个新的概念

>ideal viewport

当没有retina屏的时候，ideal viewport就等于屏幕上实际的物理像素数目，但当屏幕分辨率提高的时候，ideal viewport的尺寸还是保持不变

也就是说一个iPhone 4S无论是不是retina屏，ideal viewport都时一样的尺寸320x480

另一个要注意的是，所有缩放都是以ideal viewport为基准计算的

>visual viewport width = ideal viewport width / zoom factor

>zoom factor = ideal viewport width / visual viewport width

###我们依然要讨论几个值

`document.documentElement.clientWidth/Height`获取的是layout viewport的css像素尺寸

`window.innerWidth/Height`获取的是visual viewport的css像素尺寸

`screen.width/height`没有人关心的屏幕设备像素

`window.pageX/YOffset`现在就代表visual viewport相对于layout viewport的偏移量

`document.documentElement.offsetWidth/Height`获取html元素的css像素尺寸

###media条件判断

同样是两队值`width/height`判断layout viewport的css像素尺寸；`device-width/height`判断screen的设备尺寸

##应用

###meta viewport

说了这么多，其实真正在开发中用到的可能也就一个标签

`<meta name="viewport" content="initial-scale=1.0,width=device-width,user-scalable=0,maximum-scale=1.0"/>`

这一句应该经常被拷贝到页面的head里面，尽管很多人并不知道这些东西是什么，but it just workt!

这个标签允许我们指定layout viewport的宽度与缩放。

`width`

前面说过，移动端的layout viewport是由浏览器设定的，没有统一的标准。使用这个属性可以指定宽度，还有一个特殊值`device-width`可以将宽度设定为ideal viewport的宽度（稍后讲到这是什么）

`initial-scale`

前面又说过，多数浏览器会默认在初始化的时候将页面缩放到layout viewport刚好装进屏幕的程度，用这个属性会指定初始的缩放比例*和*layout viewport的大小

它首先会将初始的zoom设定成你指定的值（根据ideal vieport计算）用来生成visual viewport；然后，将layout viewport设定成这个visual vieport的尺寸。

实际上很难理解为什么要这么做，PPK同学在文中怒喷了一句 

>“completely fucking batshit insane.”

但这个属性就是这么干的，更复杂的情况是设置了width与initial-scale属性，实际上浏览器会取比较大的值做layout viewport的大小

`minimum-scale`

`maximum-scale`

指定最大最小缩放

`user-scalable`

指定用户是否可以缩放，0表示不允许

当然，上面说的都是一些美好的理想（或者不那么美好），但事实上要糟糕得多，关于兼容性问题，[传送门part3](http://www.quirksmode.org/mobile/metaviewport/)中做了详细的说明。

总而言之，只要不是特别特殊的需求，其实用不到这些奇奇怪怪的知识，只要把万能meta贴过来就可以，谢谢你浪费宝贵的时间看这篇没什么卵用的东西

`<meta name="viewport" content="initial-scale=1.0,width=device-width,user-scalable=0,maximum-scale=1.0"/>`

这就是你要的meta标签↑
