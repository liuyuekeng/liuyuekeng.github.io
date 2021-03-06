---
layout: post
title: "《Redis入门指南》"
date: 2016-07-16 17:12:40
categories: [books, coding]
---

最近经常见到redis的身影，但对它的了解仅限于写个值进去然后读出来的程度，
为什么redis这么受欢迎，它的优势是什么，它是怎么做的，这些都让我很好奇。
所以前阵子搞了本《redis入门指南》，书的内容基本就是介绍了redis的各种存储类型，
以及主要的命令都有哪些，顺便用一个简单的博客系统作为例子讲解。
作为一本介绍性的书还不错，看完对redis也有大致的了解了，就是用博客系统来做例子总感觉有点牵强。

redis的意思是远程字典服务，remote dictionary service，
最显著的特点是，基于内存的存储&非关系型数据库。
基于内存使他够快，非关系型的存储结构则可以理解为一个大字典，正如他的名字一样。

redis支持的数据类型有简单的字符串，散列，还有类似双向队列的列表（常被用来做任务队列），
可以做集合运算的集合，排序的有序集合。这些类型的支持让许多操作非常简便。
redis还支持事务，持久化，过期（所以经常被拿来做缓存），集群（单线程模型再也不是问题）。

整体上给人的感觉是轻量，敏捷，多样的存储类型比较灵活，在小项目中应用起来能节约不少工作量。
号称是一个数据库，但感觉上却多少有点跑偏（实际应用上也更多用来做缓存系统和任务队列）。
尽管对数据的持久化有所支持，毕竟不如文件存储那样稳健。
总之是一个很精巧的东西，用在合适的地方（比如各种运营活动）确实是一件利器。
