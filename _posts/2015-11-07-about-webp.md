---
layout: post
title:  "关于webp"
date:   2015-11-7 14:29:00
categories: coding
---

##webp是什么

google推出的一种图片格式，一开始与jpg一样，是一种有损压缩的图片格式。
后来又支持了无损压缩和透明色功能。

webp的压缩算法派生自视频编码VP8。
google号称编码后体积是jpg的55%，但是编码运算时间会比jpg长八倍。
以cpu运算，换体积大小，在现在运算资源越来越便宜，网络却一直是瓶颈。
这种交易是相当划算的。

##webp的支持情况

相对来说还是一种较新的图片格式，支持程度可以参考
![webp supported](/images/webpSupported.png)

[can I use](http://caniuse.com/#search=webp)

需要注意的是，安卓4浏览器开始支持webp格式，4.3才支持无损压缩和透明色
在安卓4.0的机子上试过，系统浏览器支持webp格式，但透明色会变成白色

##试水webp

公司的一个页面首屏加载一共24个icon，原来使用的jpg，现在需要支持透明底，
统计了一下用各种格式的图片24个icon的体积(图片请求content-length)

* jpg 205k
* png 854k
* webp 80k (无损)

webp压缩效果拔群，大概是jpg的60%，在低速网络下性能提升还是挺可观的。

##webp支持判断

一个是在服务端通过HTTP请求头判断，浏览器发送图片请求时会带上Accept头。
通过检测Acdept头判断客户端时候支持webp格式。
这有一个局限，因为只有在图片请求中才会带图片的accept头，
如果图片链接需要在之前就在模板中拼好，就没办法做了。

另一个是在客户端判断，这个方法来自一个浏览器特性检测库[Modrnizr](https://modernizr.com/)

    var test = 'data:image/webp;base64,UklGRiQAAABXRUJQVlA4IBgAAAAwAQCdASoBAAEAAwA0JaQAA3AA/vuUAAA=';
    function addResult(event) {
        var result = event && event.type === 'load' ? image.width == 1 : false;
        console.log(result);
    }
    var imge = new Image();
    image.onerror = addResult;
    image.onload = addResult;
    image.src = test;

大概的思路就是这样子，详细的代码可以到github上关注这个项目。
[github地址](https://github.com/Modernizr/Modernizr)
这是一个很棒的特征检测库。

能力检测放在客户端做是最准确的，而且代码量也很少，建议使用js做判断。
