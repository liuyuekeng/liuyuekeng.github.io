---
layout: post
title: "web页面调起app实践二"
date: 2016-07-21 15:07:51
categories: coding
---

系列传送门

[关于web页面调起APP](/coding/2016/06/12/app-invoke-from-website.html)

[web页面调起app实践一](/coding/2016/06/14/app-invoke-from-website-exp.html)

[web页面调起app实践二](/coding/2016/07/21/app-invoke-from-website-exp2.html)

上一篇中讨论了利用页面可见性判断是否调起成功，并实现了回退的逻辑。
但正如往常一样，理论总是丰满，现实却总是骨感。
使用visibility来进行回退的方案在实际应用并不那么顺利，
问题并不在属性的支持上，而是出现在这个定时器到底要定多久上。

正如我们测试中看到的，UC浏览器和QQ浏览器在web页面试图调起的时候会出现一个弹框提示，
而这个弹框，却不会阻塞js的运行，用户阅读+操作弹窗的时间，也会被记入定时器中。
如果时间太短，可能用用户还没阅读完弹窗内容，就已经被判定为调起失败执行了回退逻辑；
如果时间太长，又会让用户等待太久，可能还没有执行回退逻辑用户就不耐烦关闭页面了。

撇开弹窗不谈，就算是正常的调起流程，也不是每一次都花费相同的时间。
所以，无论这个时间设定成什么值，总会在一些场景下造成伤害——要么同时调起+回退，要么让用户陷入不知所谓的等待。

## 截获http/https

我们先来看一下这个链接

    <a href="intent://www.youtube.com/watch?v=dQw4w9WgXcQ#Intent;scheme=http;package=com.google.android.youtube;end">
        http://www.youtube.com/watch?v=dQw4w9WgXcQ
    </a>

链接的形式是之前提到过的intent scheme，但Intent中声明的scheme却是我们最熟悉的http，还声明了youtube的包名作为接收者。
尝试一番之后，我们就会发现，如果手机中安装了youtube，则会调起应用；
如果手机中并没有安装youtube，就不会有反应。

虽然这个链接没有解决我们的问题，但是指明了一个方向。
如果我们使用一个能同时被浏览器与调起目标应用拦截的隐式intent，不指明接收的包名。
那么一个正常的room就会弹出一个选择框，列出所有能够handle这个intent的应用让我们选择。
当调起目标应用不存在的时候，至少还能在浏览器中打开这个页面。
然后就可以用这个页面来做回退处理:)

## 残酷的现实

理论总是丰满，现实却总是骨感，尤其是国内的环境。
so，我们来看看到底有多残酷吧。

首先准备一个测试页面

`//example/path?id=123`，

再准备一个测试的包，其中添加测试用的intent-filter

    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <data android:scheme="http" />
        <data android:scheme="https" />
        <data android:host="example" />
        <data android:pathPrefix="/app" />
    </intent-filter>

直接访问测试页面会发现，国内的浏览器基本上都不会把http/https://的请求抛出来，
而是默默地自己处理，所以测试包接收不到调起请求。（为什么要这样互相伤害）

所以我们只能使用兼容性稍差的Intent Scheme方式，跳转到

`intent://example/page?id=123#Intent;scheme=http;end`

这样的请求，虽然和直接发送http请求理论上是一样的，但暂时还没有被国内各种坑爹浏览器拦截。

但是即使绕过了浏览器，一些国内的产商还会使用自己定制的ROM，
而这些ROM很可能并不按照标准的安卓系统来实现，so，对几个大的浏览器和手机厂商进行了测试。

    1. 华为的机子可能对http/https的intent做了特殊处理，并不会让我们的测试app接收到这个intent
    2. 小米的情况比较复杂，有些机型可以支持，有些机型则不可以（如红米note）
    2. UC浏览器实现上可能有某些bug，发送intent的时候会让用户选择‘本次允许’或者‘始终允许’，
       但选择‘始终允许’之后就无反应了，而且后续发送不会再弹出选择直接无反应。
    3. 如上一篇所说，百度浏览器可能实现了某些白名单机制，只允许某些白名单上的scheme发送，比如‘weixin://'，
       但’intent://'或者'qichebaojia://'之类的则不允许

情况不是很乐观，如果只是单纯不抛出intent还好，最坏的是处理不了跳转错误页的情况，体验实在不可接受。
结论就是，需要判断手机厂商以及浏览器外壳，再决定是否使用这种方案。

而通过UA判断国内这些厂商和浏览器，简直就是一场噩梦，可以另外开一篇文章来写了。
这里简短地说一下，github上面高星的ua判断库如[ua-aprse](https://github.com/tobie/ua-parser)，
对国内这些比较混乱的情况支持不够，而且大多比较重，只适合离线分析。
国内的有一个百度fex的开源项目[ua-device](https://github.com/fex-team/ua-device)，
对国内这些非标准的情况做了许多特殊处理考虑得比较全面（使用爬虫获取手机型号也是蛮有意思的思路），
但最后一次更新远在4个月前，似乎没人维护了。
而随着越来越多手机出现&浏览器更新，UA解析这种事情如果没有持续维护基本就渐渐失效了。
庆幸的是，只是做是否使用intent scheme方式的判断的话，并不需要太过精确，
可以自己简单地写一些正则判断一下。

调研了这么久，似乎没有一个通用的靠谱的方法能够实现我这个看似简单的需求，多少有些无奈。
真心期望大家都能够按照标准做出好用的产品，而不是各怀鬼胎地到处下绊子。
and买手机还是选三星的好，外国公司始终规范一些。
