---
layout: post
title:  "Node.js 4.0"
date:   2015-10-29 00:30:00
categories: Node
---

#突然间就4.0了

前阵子一不留神Node版本号就直接跳到4.0了，两帮人不打架了搞合体确实是普喜大奔的好事。但究竟搞了些什么并不清楚，下面是官方的文档的
[传送门](https://nodejs.org/en/blog/release/v4.0.0/?utm_source=jsgroup_weibo)

##重大变更
***
先贴个[链接](https://github.com/nodejs/LTS/wiki/Breaking-changes-between-v0.12-and-next-LTS-release)

大概看了一眼，合并的时候砍了些东西，修了点bug，版本号爬的这么快实际上变动感觉不是太大，就是以前碰巧用到这些的就要改掉啦~但如果不是新项目一般也不会这么激进地选择升级吧。

最大的升级应该就是v8引擎的升级，以及对此做的兼容工作，V8引擎升级到了v4.5，和chrome上用的一个版本，许多ES6的新特性可以支持了。当然ES6的特性不是本文的重点，可以移步到
[阮一峰的ECMAScript 6入门](http://es6.ruanyifeng.com/)

##支持平台与后续计划
***
ARM处理器支持，多平台测试集群支持Linux变种, OS X, Windows, FreeBSD and SmartOS，准备要大干一场的样子。

还要计划支持LTS和定期发布，但4.X版本不会有什么大改，而是提供18个月的LTS，消化v8的升级和兼容问题。
十月切出5.X分支做新功能的开发（咦，不就是这个月吗）后续每6个月切一个新的Stable line，做迭代，做LTS。整个流程变得更有计划，不再是以前一样被一个公司把控了，LTS的支持也让大家用起Node来更有安全感，有利于Node社区的发展。

看到现在这个局面感觉还是比较欣慰的，希望以后越搞越好。
