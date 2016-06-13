---
layout: post
title: "web页面调起app测试"
date: 2016-06-13 13:44:46
categories:
---

在上一篇blog里，讨论了[调起的各种链接形式](/coding/2016/06/12/app-invoke-from-website.html)，现在来试试实际上效果如何。

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
<tr><td>chrome 50</td><td>调起应用</td><td>调起应用</td><td>调起应用</td><td>无反应</td><td>无反应</td><td>跳转到fallback地址</td></tr>
<tr><td>手机百度 7.3.1</td><td>调起应用</td><td>调起应用</td><td>调起应用</td><td>无反应</td>无反应<td></td><td>无反应</td></tr>
<tr><td>QQ浏览器 6.7.1</td><td>提示"打开{应用名称}"，允许后可调起应用</td><td>提示"打开{应用名称}"，允许后可调起应用</td><td>提示"打开{应用名称}"，允许后可调起应用</td><td>无反应</td><td>无反应</td><td>无反应</td></tr>
<tr><td>UC浏览器 10.10</td><td>提示打开外部应用，允许后可调起应用</td><td>无反应</td><td>无反应</td><td>提示打开外部应用，允许后无反应</td><td>无反应</td><td>无反应</td></tr>
<tr><td>百度浏览器 6.7.1</td><td>可调起微信，但不能调起汽车报价（白名单）</td><td>无反应</td><td>无反应</td><td>无反应</td><td>无反应</td><td>无反应</td></tr>
</tbody>
</table>

已安装应用调起的情况相对乐观，多数是支持的。
其中QQ浏览器做得最好，还会给用户显示调起应用对应的名称，让我们为他鼓掌。
但UC浏览器和百度浏览器不支持intent scheme的形式，百度浏览器甚至可能对custom scheme设置了白名单。

未安装应用调起的情况基本没戏，之后chrome支持fallback url参数，而锤子自带浏览器则会跳到空白页，比较糟糕。
