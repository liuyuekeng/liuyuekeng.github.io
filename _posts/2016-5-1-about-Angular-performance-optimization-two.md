---
layout: post
title:  "关于Angular性能优化——懒加载"
date:   2016-5-1 14:09:00
categories: coding
---

##懒加载

[上一篇](/coding/2016/04/30/about-Angular-performance-optimization-one.html) 说的是数据预加载，这一篇就来看下关于Angular做懒加载的方法。

![angular框架初始化流程](/images/about-Angular-performance-optimization3.png)

图片来自Angular的[文档](http://docs.angularjs.cn/guide/bootstrap)，
画的是Angular框架初始化的流程。
在DOMContentLoaded的时候，把ng-app标记的元素作为根，进行模块加载，依赖注入，最后编译。
使用ng-app标记来实现bootstrap是一个自动的过程，我们不用管它什么时候初始化。

另一种方式是手动初始化，用来满足程序员们更多的需求，显然懒加载就属于这个范畴。

    angular.bootstrap(document, ['myApp']);angular.element(document).ready(function() {
      angular.bootstrap(document, ['myApp']);
    });

代码来自[参考1](http://docs.angularjs.cn/guide/bootstrap)

当然 `myApp` 需要在bootstrap之前就已经准备好，
而文档中还有一句

> You should call angular.bootstrap() after you've loaded or defined your modules.
You cannot add controllers, services, directives, etc after an application bootstraps.

这看起来有点希望破灭的味道，如果所有模块都要在bootstrap之前准备好，那还谈什么懒加载。
在bootstrap之后添加不了controller的原因在于（这里就以controller为例），
bootstrap流程中执行依赖注入的时候，这个controller还不存在，
那么框架跑起来的时候，程序自然就会报错找不到这个controller了。

回想这个过程，其实只要添加controller的时候，手动再做一次依赖注入就可以了吧。
幸运的是，$controllerProvider中提供了register方法，让我们可以手动注入。

    var app = angular.module('myApp', [])
        .config(function ($controlerProvider) {
            app.loadController = $controlerProvider.register;
        });
    
    app.loadController('myController', function $scope) {...});

代码来自[参考2](http://www.slideshare.net/nirkaufman/angularjs-lazy-loading-techniques)

经过分析，我手头的站点首次访问的绝大多数都是某页面A，
那么我将页面A相关的内容以及基础内容仍按照之前的方式加载，
而将其他页面相关的内容做懒加载。

大概有几个事情要做

1. 依赖管理，保证懒加载时加载依赖，同时避免重复加载。
3. 在路由变化前执行完需要的模块加载。

关于依赖管理的事情，可以用requirejs之类的库来做，
但懒加载这件事本来就是出于性能考虑，现在反而引入一个不小的库有点顾此失彼。
我决定自己手动管理，一个页面相关的模块在build阶段就打包成为一个文件，
公用部分的代码则放在初始时加载，省略了依赖管理的过程，粒度也相对要大一些。
懒加载的触发可以放到$routeProvider的resolve里面去，保证模块加载完才做路由跳转。


参考

http://docs.angularjs.cn/guide/bootstrap

http://www.slideshare.net/nirkaufman/angularjs-lazy-loading-techniques
