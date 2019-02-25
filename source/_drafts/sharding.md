---
title: sharding
tags:
---

* [分库分表选型和实现讨论](https://juejin.im/entry/5be97374f265da611f0739dd)
  * 在哪一层？orm、代理还是其他？
  * 主流的产品是在哪一层
  * 支持多语言？支持多数据库？
  * **shardingjdbc的连接数爆炸是什么，为什么会这样？**

<!-- more -->

shardingJDBC 
* 怎么支持多语言的？自己做了很多解析引擎，比方sql解析
* 怎么支持多orm框架？

Sharding-Proxy
定位为透明化的数据库代理端

Sharding-Sidecar 
定位为Kubernetes或Mesos的云原生数据库代理，待了解

Proxy类型
mycat，复写了MySQL 协议，伪装成一个mysql，做到支持多语言的客户端连进来


