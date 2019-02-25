---
title: linuxSysPerformance
tags:
---

![参考图片](https://static001.geekbang.org/resource/image/9e/7a/9ee6c1c5d88b0468af1a3280865a6b7a.png)

# load
什么含义？单位时间内，正在使用cpu和等待cpu和等待io的进程数，可以简称为活跃 -> 啥是活跃，为毛 -> 进程是R(Running或Runnable)或者不可中断(D)的，因为load看到是系统的繁忙程度，而不是cpu使用率 -> **超过70%算负载过高，比方单cpu，超过0.7**

看评论 
htop比较现实，忽略里面用的分析工具 
分3情况 cpu密集 io密集 大量进程的情况 
负载计算的代码网上有些文章 -> 逻辑**线程**数？ 

<!-- more -->
