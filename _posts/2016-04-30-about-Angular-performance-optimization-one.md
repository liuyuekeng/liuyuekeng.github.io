---
layout: post
title:  "关于Angular性能优化——预加载"
date:   2016-4-30 14:41:00
categories: coding
---

##数据预加载

当单页应用越来越复杂的时候，首次加载时间可能就会超出忍受的范围。
手头的angular项目首次加载的首屏渲染平均耗时已经达到4s左右，多数用户的访问时间集中在2-5秒之间。
是时候要考虑一下性能的问题了。

思路分为两个方面，一是数据预加载，二是js懒加载，这一篇先来说说数据预加载的事情。

单页应用首次展示的流程应该是页面加载->框架初始化->数据请求->模板渲染。
而实际上一个首次加载请求过来的时候，如果能加上一个标记告诉服务端需要些什么数据，
服务端在返回页面的时候把数据捎带回来，就能够省下一个数据请求的时间（一来一回能省一秒）

![请求流程](/images/about-Angular-performance-optimization1.png)

相比起react的服务端渲染机制，数据捎带的方案确实low了不少，但胜在实现起来比较简单方便。

###数据标记

实际上不需要显示地添加一个数据标记字段，改变URL的形式就可以做到这一点。
站点在之前使用的是默认的哈希形式，因为传统意义上url地址改变意味着页面的跳转。
而单页应用始终是在一个页面内，所以用哈希值来做页内的路由控制顺理成章。
HTML5的history api则提供了在不刷新页面的前提下显示改变浏览器地址栏中的URL的能力。
也就是说我们可以摆脱别扭的哈希，使用看起来舒服得多的传统URL。
而且这些事情angular框架提供了支持，不需要我们自己操心。

![hashbang_vs_regular_url](/images/about-Angular-performance-optimization2.jpg)

图片来自angular的[文档](http://docs.angularjs.cn/)。
url中的哈希值只在客户端使用，发送请求的时候并不会带给服务端。
所以对于服务端来说，a/#/item 和 a/#/list 两个不同的页面，到服务端的请求是没有任何的区别的，
都会返回整一个单页应用，然后客户端的js再根据哈希的值来决定请求哪些数据渲染哪个页面。
而将$location设置为HTML5 Modes形式后，两个页面的请求就对应地变成 a/item 和 a/list。
这两个请求对于服务端来说是可以区分的，也即是可以通过这样形式的URL告诉服务端，我们需要什么数据。

客户端需要做的事情比较零散。
首先是配置location，在app的config流程里将location设置为html5 modes。
然后排查所有的 a 标签中的 href 跳转地址和js代码中的跳转。
前者修改成为普通url的形式，后者使用$location.url做跳转。

服务端需要做的是修改路由，以前只要接受 /a 的请求，
现在则需要对 a/item 和 a/list 等都做出响应。
我们先简单地对 /a 下的请求，都做与 /a 相同的返回，即返回页面主体。

到此为止，站点整体摆脱了别扭的哈希，同时把首次加载要请求的页面告诉了服务端

###数据捎带

数据标记带到服务端之后，服务端按照业务逻辑获取到对应的数据，接下来就应该把数据带回客户端了。
这里选用的是比较老派的方式，在入口的html中加入一个textarea，然后将json_encode过的数据放到textarea中。
这样子数据就捎带回了客户端。

    <?php if($preloadData): ?>
    <textarea id="preloaddata" style="display:none;"><?php echo $preloadData?></textarea>
    <?php endif; ?>

###初始化

数据带回来之后，先判断服务端是否有捎带数据，如果没有仍需请求数据做渲染。
捎带的数据只有在首次进入的时候才有效，需要做过期的处理。
所以写了一个简单的factory来获取预加载数据

    angular.module('mysite.preload', [])                                                          
    .value('jsonData', (function () {                                                              
        var $preloadData = document.querySelector('#preloaddata');                                 
        if ($preloadData && $preloadData.innerText) {                                              
            try {                                                                                  
                var jsonData = JSON.parse($preloadData.innerText);                                 
                return jsonData;
            } catch (err) {                                                                        
                return false;
            }
        }   
        return false;
    })())
    .factory('preloadData', function (jsonData) {                                                  
        var expired = false;
        
        function get() {                                                                           
            if (expired) {                                                                         
                return false;                                                                      
            } else {
                expired = true;                                                                    
                return jsonData;                                                                   
            }
        }
    
        function cancle() {                                                                        
            expired = true;
        }
        
        return {
            get: get,
            cancle: cancle
        };
    });

用get方法获取并清除预加载数据，同时在$routeChangeSuccess清除预加载数据。

愉快地用起来

    $routeProvider.when('/item', {
        controllor: 'xxx',
        templateUrl: 'xxx',
        resolve: function (itemSrv, preloadData) {
            return preloadData.get() || itemSrv.get();
        }
    });

优化结果数据待补充。
