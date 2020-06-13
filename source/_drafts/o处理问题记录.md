---
title: o处理问题记录
tags:
---

业务启动脚本里用jps判断进程存不存在  jps报（process information unavailable）导致判断有问题 进程起了多个 
[处理方案](https://stackoverflow.com/questions/5271907/jps-process-information-unavailable-jconsole-and-jvisualvm-not-working)

<!-- more -->

2020.05.23 jdk11 mmap文件句柄没释放
现象: df -h显示占用了很大空间  到目录下`du -sh *`实际使用没那么大 
