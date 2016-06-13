---
layout: post
title:  "关于Angular性能监控"
date:   2016-3-28 12:20:00
categories: coding
---

之前一直陷在业务需求里，写码这件事情就像嚼口香糖一样，变得越来越乏味。
幸运的是现在产品相对稳定些，可以开始考虑一些优化的事情，而着一切就要从性能监控搞起。

项目嘛是一个移动端的站点，使用Angular框架，
也就是说是一个移动端的单页应用，所以和传统的应用稍微有些区别。

1. 区分首次加载与站内跳转。
与传统站点相比，单页应用的首次加载需要消耗较多的时间，而后续再页面内的跳转开销将非常低。
所以考虑监控的时候需要将首次加载与后续跳转区分开来。

2. 如何记录页面渲染完成的时间点。
在传统的站点中，这几乎不会引起任何疑问，但在单页应用中稍有不同。
所谓的单页应用不会有真正的页面跳转——也就是说传统的统计点有些并不适用。
取代页面跳转的是视图更新，所以需要在Angular框架更新视图的某一个合适流程进行打点。

下面是一个尝试的方案。

首先在title标签之后设立一个基准时间，

    <script>window.baseTime = new Date;</script>

再新建一个factory来处理性能统计相关的工作

    angular.module('myApp').factory('performanceMonitor', function () {
        var beforRoute = window.baseTime;
    });

然后就可以着手解决上面的两个问题了。第一个问题比较简单，我们可以通过记录路由事件来解决。

    angular.module('myApp').factory('performanceMonitor', function () {
        var baseTime = window.baseTime;
        var routeCount = 0;
        
        function routeStart() {
            if (routeCounte > 0) {
                beforRoute = new Date;
            }
            routeCounte ++;
        }
        
        function isFresh() {
            return  routeCount < 2;
        }
        
        return {
            routeStart: routeStart,
            isFresh: isFresh
        }
    });

每当$routeChangeStart事件触发，就调用performanceMonitor.routeStart方法。
在这个方法中记录路由触发的次数，并更新页面性能计算的基准时间——
首次加载使用一开始设立的window.baseTime，
后续的页面则设置为路由操作开始的时间点。
具体的页面可以通过isFresh方法判断是否为首次加载，从而在打点的时候区别对待。

再来看看第二个问题，为了和其他项目保持一致的标准，我使用的是首屏渲染时间——
更具体来说，是首屏所有图片加载完成的时间点。
那么问题就变成，找出首屏所有图片，再找出其中最后加载完成的那一个，
把这个时间和基准时间进行比较，就得到我们所关心的首屏渲染的效率。

    angular.module('myApp')
    .directive('viewLoad', function ($timeout) {
        return {
            restrict: 'E',
            scope: {
                callback: '='
            },
            link: function ($scope) {
                $timeout(function () {
                    $scope.callback();
                }, 0, false);
            }
        };
    });

新建一个directive来做首屏加载的标记，将这个标记插入到页面首屏的大概位置，当首屏的DOM被渲染的时候触发传进来的callback。

![screenshot](/images/about-Angular-performance-monitoring1.jpg)

这里需要解释一下$timeout的作用，可以参考下这两个问题

[Angular JS identify an digest complete event and removing # from url in angular js during viewchange](http://stackoverflow.com/questions/21138388/angular-js-identify-an-digest-complete-event-and-removing-from-url-in-angular)

[Defer angularjs watch execution after $digest (raising DOM event)](http://stackoverflow.com/questions/16066239/defer-angularjs-watch-execution-after-digest-raising-dom-event)

简单来说，就是$timeout注册的回调函数，会在digest循环，浏览器渲染之后执行。
在这个时间点能够保证要监控的DOM元素已经存在，但图片又还未加载，可以顺利地进行load事件的绑定。
至于$timeout中传递的第三个参数设置为false，是阻止$tiemout自动触发另一个新的digest，避免不必要的开销，
参见 [docs.angularjs.cn](http://docs.angularjs.cn/api/ng/service/$timeout)。
这时候只要在对应页面的controller里面向像viewLoad传递一个监控的回调就可以了。
由于策略是统计首屏图片加载完成时间，所以在performanceMonitor中再添加一个监听图片加载的方法。

    function imagesLoad (images, callback) {
        var loadPromises = [];
        angular.forEach(images, function (v, k) {
            var deferred = $q.defer();
            angular.element(v).one('load', function () {
                deferred.resolve(k);
                setTimeout(function () {
                    deferred.reject(k);
                }, 10000);
            });
            loadPromises.push(deferred.promise);
        });
        $q.all(loadPromises).then(function () {
            callback();
        })
    }

将要监听的HTML images element数组和回调传入函数，当所有图片都加载完成之后触发回调函数执行。
这里有一点需要注意，在单页应用中没有传统的页面跳转，
如果在页面中来回跳转，已经被访问过的图片对象会被缓存，不会触发onload的事件。
也就意味着这些deferred可能永远不会被解决，逐渐积累有导致内存泄漏的风险。
所以这里增加了一个超时的机制，让deferred在超时后直接reject来规避这个风险。

当然，引入超时机制只是一个委婉的方法，问题的根本在于，我们能否在拿到一个image element的时候，判断它是否已经加载完毕。
一番查找发现image element确实有一个API叫做currentSrc，支持度比较低，但确实是我们问题的答案。

![screenshot](/images/about-Angular-performance-monitoring2.jpg)

参见文档 [developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/API/HTMLImageElement)

那么再修改一下imagesLoad函数，增加对currentSrc的支持。

    function imagesLoad (images, callback) {
        var loadPromises = [];
        angular.forEach(images, function (v, k) {
            var deferred = $q.defer();
            if (v.currentSrc) {
                deferred.resolve(k);
            } else {
                angular.element(v).one('load', function () {
                    deferred.resolve(k);
                    setTimeout(function () {
                        deferred.reject(k);
                    }, 10000);
                });
            }
            loadPromises.push(deferred.promise);
        });
        $q.all(loadPromises).then(function () {
            callback();
        })
    }

直到这里，一开始提出的两个问题已经得到解决，撒花~~
