---
title: dubbo
tags:
---

# 异步调用

* 老版本的异步调用很弱 Future get 方法其实也是阻塞
* 老版本dubbo的ResponseFuture也很弱，没有java8的completeFuture的api强大，比方如多个Future协同工作 [全异步调用链](http://dubbo.apache.org/zh-cn/blog/dubbo-new-async.html)
* 异步是Consumer端的异步，provider端不支持
* A -> B 是异步 B的处理过程里本来是同步调的C，然鹅因为 A调B的时候在context里设置了异步的标志，导致B调C也变成了异步，然后会取不到结果 [解决方案](https://blog.csdn.net/windrui/article/details/52150345)  [包装过的异步调用](https://www.wenji8.com/p/1abLjG6.html)
* jim-framework 和 黄勇及阿帆卢的版本对比，做了哪些改进

<!-- more -->

# 版本、接口设计问题

# 责任链 上下文 threadLocal

# SPI
* 原生SPI的梳理 [比较详细](https://zhuanlan.zhihu.com/p/28909673)
{% asset_img spi.png  %}
**接口属于实现方的情况，就是我们经常说的api，组织上位于实现方所在的包中
当接口属于调用方时，我们就将其称为spi** 

* [dubbo如何做到高度可扩展](http://dubbo.apache.org/zh-cn/blog/introduction-to-dubbo-spi.html)
* [官方源码，特别详细，包含warpper](http://dubbo.apache.org/zh-cn/blog/introduction-to-dubbo-spi-2.html) 
* 原生的spi有什么问题
  * 扩展如果依赖其他的扩展，做不到自动注入和装配。不提供类似于Spring的IOC和AOP功能
    * 怎么解决  Adaptive注解 [链接](https://juejin.im/post/5acee93b51882555731c82ba)
* ExtentionLoader
* wrapper方面

# 限流
客户端 actives="10"参数  支持方法级的限制  ActiveLimitFilter实现
服务端  executes="10"参数 ExecuteLimitFilter 原理是java信号量做控制 tryAcquire取不到马上返回false 抛异常
TpsLimitFilter 服务端 支持 方法级的 tps限流 实现了滑动窗口做精确控制



