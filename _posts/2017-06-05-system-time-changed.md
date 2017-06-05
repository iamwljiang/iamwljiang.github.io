---
layout: post
title:  "定位系统时间跳变"
categories: linux
tags:  linux shell 
author: iamwljiang
---

* content
{:toc}

# 系统时间跳变

本文将介绍linux上系统时间跳变的定位过程，换个角度来思考如何处理这类的问题。

## 问题现象
centos6.5系统下，在测试环境中出现了时间跳变，影响了部分业务的正常运行。通过date可以看到系统时间确实和当前时间出现了几个小时的差异，并且修正回来之后，间隔一段时间又复现。

## 处理思路
* 查看messages
* 查看hwclock是否硬件时钟也出现了跳变
* 检查系统上的ntp服务是否异常
* 检查系统时区
* 替换系统命令，跟踪调用进程


## 处理方法与结果

前面几项排查无异常，只能对时间相关的进程做修改，跟踪调用过程。

修改方法：重命名程序，加入同名的脚本，内部记录调用该命令的父进程信息，以及执行的命令，并打印日志信息，便于排查。日志需要注意回滚。


1.date命令



```bash

time=`/bin/date.real +"%F %T"`
sec=`/bin/date.real +%s`
file="/tmp/jwl.date.log"
bak="${file}_bak.${sec}"
echo "$time  process id($PPID)  ==`ps -p $PPID|tail -n 1`== call date cmd:[$0 $@]" >> $file
/bin/date.real $@
size=`stat -c %s $file`
if [ $size -ge 50000000 ];then
    count=`grep -c "\-s" $file`
		if [ $count -ge 1 ];then
				mv $file $bak
		else
				rm $file
		fi
fi
```


2.ntpdate命令

类似同上操作

3.hwclock命令

类似同上操作


**结果与验证**

最终可以确定是确实有一个进程在代码中直接修改了系统的时间，并调用了hwclock -w把系统时间反写到硬件时钟。

根据进程号和进程名，定位到进程的bin程序目录，`strings 进程名 | grep hwlock` 确实可以看到代码中有调用。