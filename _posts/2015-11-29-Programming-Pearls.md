---
layout: post
title:  "《编程珠玑》"
date:   2015-11-29 14:24:00
categories: books
---

在同事的书桌上看到这本书，想着借来打发周末的时间。
翻开第一章我才知道，如果要认真回答书里提出的问题，一个周末是远远不够的，于是自己买了一本。

虽然主题是编程，但内容实际上更宽泛（通用）一些，作者想表达的是解决问题的智慧。
每一章中先阐述一个思想，随后抛出一系列问题引导读者自己去思考。

作为一个比较懒的人，跳过问题直接看答案的冲动不可谓不强烈。
但静下心来慢慢解题就会发现作者藏在题目后面想要告诉你道理。
不要着急前进，而是当做一种周末的消遣，
就像玩解密游戏一样，通过思考解决有趣的问题，大概更能够体会到这本书的乐趣。

-----------

###我的答案

####1.1

我选择用js写一个大数组然后直接调用sort方法

    var fs = require('fs');

    var output = 'start: ';
    var start = (new Date()).getTime();
    output += start;
    fs.readFile('./input', 'utf8', function (err, data) {
        if (!err) {
            var array = data.split('\n');
            array.pop();
            array = array.sort(function(a, b) {
                return (a - b);
            });
            var res = array.join('\n') + '\n';
            fs.writeFile('./out', res, function () {
                var end = (new Date()).getTime();
                output += (' end: ' + end + ' coust: ' + (end - start));
                console.log(output);
            });
        }
    });

####1.2

一开始看到并不知道题目说的什么意思。
看了答案以及别人的解释，才明白是用32位int型数组拼起来做位向量。
我一直以为是直接拉一块内存出来然后直接进行操作的囧。
毕竟写js从来没有自己手动管理过内存，对这种没什么概念。

由于js中也没什么整形，就直接用buffer来写，每一个位8bit

    var buffer;
    var len = 50;
    var bitStep = 8;
    var shift = 3;
    var marsk = 7;

    function init() {
        var bufferLen = len / bitStep + 1;
        buffer = new Buffer(bufferLen);
        buffer.fill(0);
    }

    function set(i) {
        buffer[i >> shift] |= (1 << (i & marsk))
    }

    function del(i) {
        buffer[i >> shift] &= ~(1 << (i & marsk))
    }

    function test(i) {
        console.log(buffer[i >> shift] & (1 << (i & marsk)));
    }

    init();
    console.log(buffer);
    for (var i = 0; i < 10; i ++) {
        set(i);
    }
    for (var j = 0; j < 20; j ++) {
        test(j);
    }
    console.log(buffer);

