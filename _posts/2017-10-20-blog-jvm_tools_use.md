---
layout: post
title: "JVM相关工具使用"
description: ""
category: 技术分享
tags: [blog]
---
{% include JB/setup %}
# JVM相关工具使用
---

1. jstat -gcutil 1000   查看gc情况 1s钟刷新一次
2. jmap -p pid 查看内存使用情况
3. jstack pid 打印线程栈
4. top 查看实时机器性能
5. htop top的升级版

### CPU占用过高排查方法

1. top/ps 查看哪些进程占用CPU比较高，由于排查的是基于java的web应用，所以一般都是command为java的进程，复制进程ID。

2. top -H -p pid 查看某进程（上一步取到的占高CPU的进程ID）的所以线程。找出占用CPU最高的线程ID

3. printf "%x\n" spid 获取线程八进制值。

4. jstack打印线程堆栈，堆栈中搜索上一步获取的八进制线程号，即可定位到具体的问题代码。
