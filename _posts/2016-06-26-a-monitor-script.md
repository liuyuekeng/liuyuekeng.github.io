---
layout: post
title: "监控脚本——日志记录&生成静态页面"
date: 2016-06-26 21:05:05
categories: coding
---

项目中遇到一个问题，项目的部分逻辑是依靠一个平台上的配置文件来驱动的。
在站点日志中发现一些异常，与这部分逻辑相关，
逻辑没有动过，但出现异常的时间只有两天，后来又自己恢复了，初步怀疑与读取配置或者配置平台有关，
所以尝试加一个监控。

首先，写一个读取配置平台对应数据的web接口，方便随时查看配置数据。
然后只要写一个脚本，定时访问接口，并作为日志文件保存起来

新建一个shell脚本，把访问web接口得到的结果加上时间信息，用|||分割，按行写入日志文件中。

    #! /bin/sh
    date=$(date +%Y%m%d%H)
    sedcmd=s/^/$date\|\|\|/
    logFile=./xxx
    curl -s http://aaa |
        sed -e $sedcmd -e 's/$/\n/' >> $logFile

为了防止日志文件无限增大，增加一个数量限制，假设每小时记录一条日志，保存最近一周，
也就是24 x 7 = 168条，增加如下代码。计算行数，超出限制则取出最新的部分。

    limit=168
    count=`wc -l $logFile | awk '{printf $1}'`
    if [ $count -gt $limit ]
    then
        tail -n $limit $logFile > $logFile
    fi

当然，直接看日志文件并不够酷，我也不乐意别人登到我的机子上来。
我们可以根据这个日志文件生成一个简单的静态页面供他人&自己访问。
其他的部分可以简陋一些，最重要的一点是，希望把“与前一条相比发生变化”的日志进行飘红。
这样查看的时候就可以轻易地看到变化点。
第一步是在日志文件中把有diff的日志行标记出来，这里使用@@在行首进行标记。

    if [ $count -gt 1 ]
    then
        eval $(tail -2 $logFile | awk -v 'FS=[|][|][|]' '{printf $2"^Z"}' |
            awk -v 'FS=^Z' '{printf "tmp1="$1"\ntmp2="$2}')
        if [ $tmp1 != $tmp2 ]
        then
            sed -i -e '$s/^/@@@/' $logFile
        fi
    fi

代码的逻辑就是拿出文件后两行=》去掉时间信息=》赋值到两个变量=》比较是否相等
=》不相等就将最后一行加上@@@的前缀。
日志中的标记妥当之后，我们就可以用日志来生成静态页面啦。
首先是把文件逆序一下，让最新的日志排在最前，然后使用正则替换生成html，
对有@@@标记的进行飘红。

    tac $logFile | sed -e 's/^\([^@].*\)$/\<span\>\1\<\/span\>/g'
        -e 's/^@@@\(.*\)$/\<span style="color:red"\>\1\<\/span\>/g'
        -e 's/$/\<br\>\<br\>/' $logFile > $tplFile

通过静态页面查看日志比看文件强多啦，但还是不够酷。
我并不会经常来看监控，所以希望当日志变动的时候，能够发邮件。
在判断 $tmp1 与 $tmp2 不相等的时候，把日志发到自己的邮箱，顺便附上日志页面链接。

    cat | mail -s '【配置报警】' 1234@xxx.com << EOF
    befor:
    $tmp1
    after:
    $tmp2
    log detail:
    http://xxxxx
    EOF>>

接下来只要让它定时执行就可以了

`crontab -e`

加入定时任务，每小时执行一次

    0 * * * * /home/xxx.sh &> /dev/null

这里有几个问题，首先是脚本中的相对路径需要修改，
crontab执行定时任务的时候并不会进入到脚本所在的目录，而是在用户目录下执行，
也就是上面代码中logFile的定义需要修改。
同理，环境变量等也会有所不同，需要在crontab配置中指定，或者在代码中声明。
比如发邮件带了中文，需要指定字符集
`export LANG=en_US.UTF-8`
另一点是权限问题，需要给刚刚的脚本文件添加执行权限。

`chmod +x xxx.sh`

可以通过查看/var/log/cron判断定时脚本是否执行，
而执行结果默认会给你当前用户发个邮件，我就是在邮件里看到权限问题的。
至于上面定时任务中的`&> /dev/null`就是在调试结束后，要把这烦人的邮件去掉。

其实shell并不适合用来写这种具体的逻辑，估计两个星期后我就已经看不懂自己写的啥了。
但shell确实是一个精巧的东西，作为一个调度工具来使用还是挺顺手的。
这次就当时试试水吧，了解一下shell的工作方式，顺便也践行一下DRY原则，把这些劳力工作交给脚本。