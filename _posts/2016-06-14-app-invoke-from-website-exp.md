---
layout: post
title: "web页面调起app测试"
date: 2016-06-14 10:05:46
categories: coding
---

在上一篇blog里，讨论了[调起的各种链接形式](/coding/2016/06/12/app-invoke-from-website.html)，现在来试试实际上效果如何。

### intent兼容性测试

<a href="weixin://">微信custom scheme</a>
<br><a href="intent://#Intent;scheme=weixin;end">微信intent scheme</a>
<br><a href="intent://#Intent;scheme=weixin;S.browser_fallback_url=https%3A%2F%2Fwww.baidu.com%2F;end">微信intent scheme with fallback</a>
<br><a href="qichebaojia://carserial?id=3544">汽车报价大全custom scheme</a>
<br><a href="intent://carserial?id=3544#Intent;scheme=qichebaojia;end">汽车报价大全intent scheme</a>
<br><a href="intent://carserial?id=3544#Intent;scheme=qichebaojia;S.browser_fallback_url=https%3A%2F%2Fwww.baidu.com%2F;end">汽车报价大全intent scheme with fallback</a>

<table>
<thead>
<tr><th rowspan="2">浏览器</th><th colspan="3">已安装</th><th colspan="3">未安装</th></tr>
<tr><th>coustom scheme</th><th>intent scheme</th><th>intent scheme with fallback</th><th>coustom scheme</th><th>intent scheme</th><th>intent scheme with fallback</th></tr>
</thead>
<tbody>
<tr><td>锤子自带浏览器</td><td>调起应用</td><td>调起应用</td><td>调起应用</td><td>跳转到错误页</td><td>跳转到错误页</td><td>跳转到错误页</td></tr>
<tr><td>小米自带浏览器</td><td>调起应用</td><td>调起应用</td><td>调起应用</td><td>无反应</td><td>无反应</td><td>无反应</td></tr>
<tr><td>chrome 50</td><td>调起应用</td><td>调起应用</td><td>调起应用</td><td>无反应</td><td>无反应</td><td>跳转到fallback地址</td></tr>
<tr><td>手机百度 7.3.1</td><td>调起应用</td><td>调起应用</td><td>调起应用</td><td>无反应</td>无反应<td></td><td>无反应</td></tr>
<tr><td>QQ浏览器 6.7.1</td><td>提示"打开{应用名称}"，允许后可调起应用</td><td>提示"打开{应用名称}"，允许后可调起应用</td><td>提示"打开{应用名称}"，允许后可调起应用</td><td>无反应</td><td>无反应</td><td>无反应</td></tr>
<tr><td>UC浏览器 10.10</td><td>提示打开外部应用，允许后可调起应用</td><td>无反应</td><td>无反应</td><td>提示打开外部应用，允许后无反应</td><td>无反应</td><td>无反应</td></tr>
<tr><td>百度浏览器 6.7.1</td><td>可调起微信，但不能调起汽车报价（白名单）</td><td>无反应</td><td>无反应</td><td>无反应</td><td>无反应</td><td>无反应</td></tr>
</tbody>
</table>

* 已安装应用调起的情况相对乐观，多数是支持的。
  其中QQ浏览器做得最好，还会给用户显示调起应用对应的名称，让我们为他鼓掌。
  但UC浏览器和百度浏览器不支持intent scheme的形式，百度浏览器甚至可能对custom scheme设置了白名单。
* 未安装应用调起的情况基本没戏，只有chrome支持fallback url参数，而锤子自带浏览器则会跳到空白页，比较糟糕。

### 如何回退？

在具体业务逻辑里，没有回退的情形很难令人接受。
我更希望实现的形式是，如果目标应用未在本地安装，将用户引导至应用的下载页面。
这样的体验更加顺畅，也能给我的应用带来新增激活，简直完美:)

靠原生的机制看来是不靠谱的，这里提出一个迂回的方法，利用visibilitychange事件来判断调起是否成功。
visibilitychange是什么东西可以参考[mozilla的文档](https://developer.mozilla.org/zh-CN/docs/Web/Events/visibilitychange)，
或者可爱的[张老师的文章](http://www.zhangxinxu.com/wordpress/?p=2790)。

假设我们本地已经安装了目标应用，那么Intent调起之后，浏览器自然就被放到后台去了。
如果未安装目标应用，Intent请求发送后多数是没有反应的。
那么就可以通过visibility是否可见来近似地判断调起是否成功。
当然，用户点击调起后立刻离开浏览器/页面等特殊情况会被误判，这也是没办法的事。

流程大概是这样，发送调起请求的同时启动一个定时器，当页面变为不可见时取消定时器，否则当定时器触发时触发对应的回退逻辑。

成功流程

![成功流程](/images/app-invoke-from-website-exp1.png)

失败流程

![失败流程](/images/app-invoke-from-website-exp2.png)

### visibilitychange支持测试

<table>
<thead>
<tr><th>浏览器</th><th>支持情况</th></tr>
</thead>
<tbody>
<tr><td>锤子自带浏览器</td><td>不支持</td></tr>
<tr><td>小米自带浏览器</td><td>支持</td></tr>
<tr><td>chrome 50</td><td>支持</td></tr>
<tr><td>手机百度 7.3.1</td><td>不支持</td></tr>
<tr><td>QQ浏览器 6.7.1</td><td>支持</td></tr>
<tr><td>UC浏览器 10.10</td><td>支持</td></tr>
<tr><td>百度浏览器 6.7.1</td><td>不支持</td></tr>
</tbody>
</table>

根据mozilla的文档，安卓原生浏览器是没有实现这个功能的，支持程度总得来看也不低了。
而且这个API也已经稳定下来了，可以预见将来应该会更广泛地被支持。
所以值得为支持该事件的浏览器实现这一套逻辑。
如何判断这个API是否存在似乎是个难题，类似[Modrnizr](https://modernizr.com/)，之类的库或者网上的文章，
都只是检测document的hidden属性或者visibilityState属性（加上前缀判断），
但似乎没有办法判断visibilitychange事件是否存在╮(╯3╰)╭

特征检测行不通，可能需要使用UA检测，对此隐隐感到有些担忧，但暂时没有更好的方案。
