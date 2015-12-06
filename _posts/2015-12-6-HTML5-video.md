---
layout: post
title:  "关于&lt;video&gt;视频播放"
date:   2015-12-6 16:18:00
categories: web
---

###video标签

&lt;viedeo&gt;是HTML5增加用来播放视频的标签。

    <video>
        <source src="movie.ogg" type="video/ogg">
        // source标签用来提供多个来源供浏览器选择
        <source src="movie.mp4" type="video/mp4; codecs=dirac, speex">
        // 除了类型，还可以指定解码器，如果不支持指定解码器，也不会播放这个源
        // MIME-type未定义浏览器会请求资源进行判断，所以能确认的情况还是加上
        
        你的破浏览器不支持这玩意儿  //video标签不被支持时显示的内容
    </video>

除了多来源，还可以使用&lt;track&gt;切换字幕，可惜支持程度不忍直视。

除了source标签和track标签，剩下的内容是在video不被支持的时候显示，可以方便地用来做回退方案。
比如把flash插件丢进去。

###属性支持与事件支持

video标签的[属性](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)
    
DOM元素提供的
[接口](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement)
足够你获取视频的各种信息
媒体元素的丰富
[事件](https://developer.mozilla.org/en-US/docs/Web/Guide/Events/Media_events)
也能让你在视频状态任何变化时得到通知。

有了这些，用js自己实现控制器并不困难。
值得注意的是有一个全屏的方法，在chrome中提示不支持使用js来触发了。
而原生控制器中的全屏在有些国产山寨浏览器中会有不同的表现。

###视频格式问题

视频编码兼容，因为专利等问题，情况堪忧
[媒体类型支持](https://developer.mozilla.org/en-US/docs/Web/HTML/Supported_media_formats)
，但总的来说如果能支持webM和mp4的话基本就ok。

webM是google推的一种格式，画质会比mp4高一些。

video元素有一个方法
[canPlayType()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLMediaElement/canPlayType)
，可以在资源加载之前判断类型是否支持提前一步做处理。
判断不了就只能等无法解析触发error事件的时候处理了

###别人怎么做

####优酷

围观了一下人家优酷，在移动端也是使用了video标签，然后自己实现了一套控制器。
UI外观也好，各种功能逻辑也好，都不使用原生的属性和方法。
控件自己实现，连poster也是自己写的一个遮罩层

视频编码采用了mp4，实现了视频分片的逻辑，避免单个视频文件太大。
所以然看一半切换文件的时候会跳一下。
播放进度也需要自己另外进行计算。

全屏的实现是把video标签放在一个容器中，宽高设为100%，
通过改变外层容器来做全屏。屏蔽滚动的事件，避免背后的页面滚动。

旋转的事件似乎是没有管，全屏之后旋转屏幕并不会重新改变视频朝向。

摒弃原生控件固然有保持自身UI一致的原因，
但应该也有国内浏览器喜欢自己乱改控件的原因。

DOM接口与丰富的事件足以让我们非常方便地实现简单视频功能，
为了避免被各种山寨浏览器坑，还是自己实现比较好

####youtube

youtube相对简单得多，直接使用了原生的控件。可能是国外的浏览器环境相对要好一些。

视频编码使用的是webM，毕竟google自家人，画质上相对会好一些。

###参考

[HTML5 video 使用介绍](https://developer.mozilla.org/en-US/docs/Web/Guide/HTML/Using_HTML5_audio_and_video)

[优酷网](http://www.youku.com/)

[youtube移动版](https://m.youtube.com/)
