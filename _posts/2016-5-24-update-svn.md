---
layout: post
title:  "SVN升级记录"
date:   2016-5-24 18:17:00
categories: coding
---

机器上的svn版本在1.6，svn文件分布在各个目录下。
使用bower获取第三方库的时候，总会把bower_components中的svn文件删除。
所以决定升级svn版本，1.7之后svn文件放在版本库的根目录下，不会再受bower的影响。

开发机系统版本也比较老，yum上的svn版本也只有1.6，所以只能手动安装。

先总结一下，svn升级1.8的时候需要依赖apr、apr-unit、serf-1.2.1(http客户端)，
[https://blog.linuxeye.com/348.html](https://blog.linuxeye.com/348.html)这篇教程写得挺好，可惜等我把坑趟平之后才看到。
下面是踩坑过程的记录。

下载svn代码
[http://subversion.apache.org/download/](http://subversion.apache.org/download/)

    ./configure

报错

    no suitable apr found

需要依赖阿帕奇，用yum安装apr-devel

    yum install apr-devel

结果yum执行报错（我到底多久没有用过yum了，现在才发现）

    It's possible that the above module doesn't match the
    current version of Python, which is:
    2.7.9 (default, Aug 24 2015, 11:16:37) 
    [GCC 4.4.6 20120305 (Red Hat 4.4.6-4)]

应该是python版本的问题

    whereis python

发现机器上装了python3.0，需要把yum指定到Python2.7。
修改`/usr/bin/yum`和`/usr/bin/yum-updatest`中的第一行

    #!/usr/bin/python  =>  #!/usr/bin/python2.7

修改后yum就能正常工作了。
接着安装apr，安装完成之后，执行

    ./configure

报错

    no suitable APRUTIL found

还少一个apr的库

    yum install apr-util-devel

安装apr-util之后，再执行

    ./configure

报错
    An appropriate version of sqlite could not be found.  We recommmend
    3.7.15.1, but require at least 3.7.12.
    Please either install a newer sqlite on this system

    or

    get the sqlite 3.7.15.1 amalgamation from:
    http://www.sqlite.org/sqlite-amalgamation-3071501.zip
    unpack the archive using unzip and rename the resulting
    directory to:
    xxxxxxx/subversion-1.8.16/sqlite-amalgamation

sqlite版本太低，yum上的sqlite版本也太低，直接按照提示下载sqlite的包放到编译目录下，再执行

    ./configure && make  && make install

安装成功

    svn --version

    svn, version 1.8.16

回到工作目录执行svn upgrade，更新代码中的svn版本，再执行

    svn update

报错

    svn: E170000: Unrecognized URL scheme for httpxxxx

没有安装http模块，只能用本地的源。
svn再1.8开始使用serf做http模块，之前使用的是neon，以serf为例

    wget http://serf.googlecode.com/files/serf-1.2.1.tar.bz2

下载解压之后

    ./configure && make && make install

重新安装svn，并指定http模块，然后重新装一次

    ./configure --with-serf=/usr/local/serf

    make && make install

完成

参考

[http://superuser.com/questions/280416/ubuntu-install-subversion-from-source-missing-apr](http://superuser.com/questions/280416/ubuntu-install-subversion-from-source-missing-apr)

[http://wiki.openkm.com/index.php/Tomcat_native_libraries](http://wiki.openkm.com/index.php/Tomcat_native_libraries)

[http://blog.csdn.net/ei__nino/article/details/8495295](http://blog.csdn.net/ei__nino/article/details/8495295)

[https://blog.linuxeye.com/348.html](https://blog.linuxeye.com/348.html)
