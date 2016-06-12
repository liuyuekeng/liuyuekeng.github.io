---
layout: post
title: "关于web页面调起APP"
date: 2016-06-12 10:51:11
categories: coding
---

工作中会遇到许多web页面要和本地APP进行交互的场景，由于隔着浏览器沙盒和安卓系统，要顺畅地实现这种通信是比较蛋疼的。
加上国内环境比较复杂，各家浏览器都打着自己的小算盘，无疑加大了通信的难度。
这里来总结下工作中接触过的几种方式（这里只讨论安卓）。

###本地server代理
我们想要web页面和本地的应用A进行交互，那么就让应用A在后台启一个server。
web页面中向这个本地的server发送HTTP请求，由本地server进行代理，与应用A进行交互。
本地server在这里扮演着一个双重角色，他即是一个HTTP服务器——接受web页面的HTTP请求，
又是一个本地service——能够通过intent与系统内的其他应用进行通讯，由他来做一个桥梁。
![native server](/images/app-invoke-from-website1.png)

    * 优势很明显，只要设计好server的接口，就可以像请求一个服务一样使用本地应用A的能力。
      而最大的优点是，这个调用对web端来说就是一个普通的请求，可以有返回值——
      也就是说可以做成功的回调和失败的回退逻辑。

    * 缺点也很严重，必须保证本地server启动才能执行通信，要维持这么一个不会挂的server就需要一些额外的流氓手段了。
      另一点是，如果不对接口进行加密走私有协议，或者没有完善的验证，这个server容易被第三方利用。
      最后的一点是，这种方式没有办法支持HTTPS，本地server没有固定域名，自然没有办法签证书，HTTPS也就无从谈起了。

这其实是一个比较名不正言不顺的方案，纯粹为了绕过浏览器沙盒加了一个本地server代理，随着HTTPS的到来也就过时了。
(其实目前有些国内浏览器还没有禁止在HTTPS环境下发送HTTP请求，但从趋势上来看也终将消亡)

###OpenURL
如果我们不想局限于与某个应用A的交互，而是想调起任意本地应用的功能，就需要一种更通用的形式。
看看上面的本地server形式，如果浏览器能够承担这个桥梁的职责，那我们就完全用不着这个server了。
换句话说，我们可以把对本地server的依赖转移到对浏览器的依赖上。

流程大概是这样的，某应用A在安装的时候，向系统注册了一个custom scheme 协议为aaa。
然后用户访问web页面的时候我们发送请求`aaa://xxx?k=v`

    var ifr = document.createElement('iframe');
    ifr.width = 0;
    ifr.height = 0;
    ifr.src = 'aaa://xxx?k=v';
    document.body.appendChild(ifr);

这样就可以调起对应的应用了。系统中其实有许多默认的协议，比如打电话`tel:1234567`，发短信`sms:1234567`等。
如果有多个应用注册了同一个协议，比如本地有两个视频播放应用A和B都注册了`video`协议，那么浏览器就会询问用户使用A还是B来执行这个调起操作。
微信，支付宝等应用也有OpenURL形式的接口提供端外浏览器中调起的能力（可以试试`weixin://`和`alipay://`）。

    * 优点是，只要APP开发者提供了对应的URL，我们就可以发送请求使用APP的功能了，没有额外的负担。
      在安卓与iOS系统中通用，不必使用两套机制。

    * 缺点是。。。如果有恶意应用通过注册协议留了个后门，那么访问对应的恶意站点可能造成安全问题。
      而且这个注册行为也不需要验证，任意应用都完全可以注册weixin前缀。
      所以这种方式以后将不被支持（可以试试，在chrome中新建这么一个iframe已经无法调起了）
      目前来说，这种方式还有另一个缺陷，我们没办法得到反馈——
      也就是说请求发出去之后我们就管不了了，APP安装了吗？调起到了APP了吗？执行成功还是失败一概不知。

关于安全验证方面，也可以参考微信，支付宝的方式。OpenURL中需要带一个加密串，这个加密串则是要通过一个接口请求获得的。
具体流程应该是：请求加密串，拼装OpenURL，发起请求，微信对加密串进行校验，执行支付，退出微信。
可以看出，验证的逻辑需要另一个验证的服务来支持，而不是OpenURL本身能够完成的工作。
![authentication server](/images/app-invoke-from-website2.png)

###Intent 

intent是安卓系统里面应用之间通信的消息对象，而浏览器也是一个应用。
那么浏览器如果支持代理Intent请求，也可以完成web页面到本地应用的通讯。
通过发送Intent scheme URL，可以实现这样的功能，简单地说来就是web页面发送类似下面这样的请求

    intent://xxx/#Intent;action=aaa;type=text/plain;S.a=a;i.b=1;end

浏览器会对这个URL进行解析，并根据这个url构造一个Intent对象，使用Intent与应用进行通讯。
所以，我们在web页面发送的并不是一个Intent，而是一个Intent scheme URL，真正发送Intent的是浏览器。
换句话说，我们还是摆脱不了“浏览器支持”这件蛋疼的事。

下面例子来自[https://paul.kinlan.me/every-browser-should-support-intent-urls/](https://paul.kinlan.me/every-browser-should-support-intent-urls/)

假设我想调起twitter，只需要发送请求

    intent:#Intent;package=com.twitter;end

实际上intent scheme的形式更为灵活，custom scheme的调起也可以支持\n
`intent://user?screen_name=paul_kinlan#Intent;scheme=twitter;end`可以实现\n
`twitter://user?screen_name=paul_kinlan`的功能。
甚至更进一步，在twitter未安装的情况下提供一个web回退地址\n
`intent://user?screen_name=paul_kinlan#Intent;scheme=twitter;S.browser_fallback_url=https%3A%2F%2Ftwitter.com%2Fpaul_kinlan;end`\n
当twitter未安装的时候，页面将跳转到`https://twitter.com/paul_kinlan`，而不会毫无反应（chrome 40）

<a href="intent://www.youtube.com/watch?v=dQw4w9WgXcQ#Intent;scheme=http;package=com.google.android.youtube;end">http://www.youtube.com/watch?v=dQw4w9WgXcQ</a>

测试链接来自https://firebase.google.com/docs/app-indexing/android/test#http-url-testing-tool

###参考

1. https://paul.kinlan.me/android-intent-fallback-detection/
2. https://paul.kinlan.me/every-browser-should-support-intent-urls/
3. https://paul.kinlan.me/launch-app-from-web-with-fallback/
4. https://firebase.google.com/docs/app-indexing/android/test#http-url-testing-tool
5. http://drops.wooyun.org/papers/2893
6. http://www.xxdafa.com/article?id=56965123b8063f1a058b456f
7. http://biancheng.dnbcw.info/shouji/390830.html
8. http://liangruijun.blog.51cto.com/3061169/634411/
