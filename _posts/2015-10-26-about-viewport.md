---
layout: post
title:  "关于viewport"
date:   2015-10-26 23:14:00
categories: web
---
#为什么关注这个
****

看到一些找前端的要求里会写要有移动端开发的经验，做了一年所谓的移动端开发，感觉和PC端的区别并没有太大。而viewport就是其中最先接触到的一点。

正巧看到一篇PPK关于viewport姿势的文章

[传送门part1](http://www.quirksmode.org/mobile/viewports.html)

[传送门part2](http://www.quirksmode.org/mobile/viewports2.html)

[传送门part3](http://www.quirksmode.org/mobile/metaviewport/)

英文的回顾起来比较费力，下面算是笔记

#正文
****

##css像素 VS 设备像素

设备像素很好理解，就是数出屏幕上有多少个像素点，可以通过screen.width/height来获取，可以理解为不变的量

css像素就是样式中操作的像素，而当zoom为100%的时候，每一个css像素刚刚好和设备像素一样大。

当你进行拉伸的时候，实际上改变的是每一个css像素的大小，而不是他们的数目，也就是说每一个css像素占用更多个设备像素。

一般来说我们并不关心设备的像素尺寸，因为毕竟我们的页面是按照css像素进行渲染的。

##viewports

视口可以认为是用来装<html>元素的容器，但视口的大小和html的大小是不等价的。

在PC端，viewport的宽高就是浏览器窗口里面能显示区域的宽高，而一个页面一般比一个屏幕要长，在移动端会更复杂一些

##讨论几个值

`document. documentElement. clientWidth/Height`可以取到viewport的css像素；

`document. documentElement. offsetWidth/Height`可以取到html元素的css像素；

`window.innerWidth/innerHeight`是浏览器窗口的css像素尺寸，与clientWidth/Height不同的是，它包含滚动条；

`window.pageX/YOffset`是页面偏移量的css像素

点击事件带的坐标有`pageX/Y`（相对于html元素计算）,`clientX/Y`（相对于viewport计算）,`screenX/Y`（相对于屏幕计算的设备像素值，没什么卵用）

##media条件判断

适配不同大小屏幕，可能会用到类似`@media all and (max-width: 400px)`之类的条件判断来重写样式。

如果用`width/height`进行条件限制，判断是是视口的尺寸（css像素），如果是用`device-width/device-height`进行条件限制，判断的是屏幕的尺寸（设备像素）。

##移动端的视口

TODO
