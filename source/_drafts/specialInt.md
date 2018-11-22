---
title: specialInt
tags:
---

* 简易RPC + Netty + ThreadLocal
  * 异步调用 调用链 先细节 再看如何实现 context threadLocal   
    * 老版本的异步调用很弱 Future get 方法其实也是阻塞
    * 老版本dubbo的ResponseFuture也很弱，没有java8的completeFuture的api强大，比方如多个Future协同工作 [全异步调用链](http://dubbo.apache.org/zh-cn/blog/dubbo-new-async.html) 
    * 异步是Consumer端的异步，provider端不支持
    * A -> B 是异步 B的处理过程里本来是同步调的C，然鹅因为 A调B的时候在context里设置了异步的标志，导致B调C也变成了异步，然后会取不到结果
    
  * SPI 的设计，比原生的java spi解决了什么问题 
  * Filter链条
  * jim-framework 和 黄勇及阿帆卢的版本对比，做了哪些改进
* 套入处理内存泄露问题 + socketio-netty + vjtools
* 用ansible搭一个高可用队列的拓扑

<!-- more -->

# 项目相关
* 数据量 
  * 10w * 2 * 5 * 30 = 3kw 月4800w
  * 年3亿 
  * 4库4表 
    * 日期分库(查询条件带日期)、料号分表
* 用shardingJDBC 处理 
  * 分库分表和读写分离整合
    * **数据怎么路由到库和表的**
    * 了解读写分离怎么路由的 AbstractRoutingDataSource
  * 双sharding key问题解决 
    * 有个关系表，每个查询先查关系表
    * 拆分卖家库和买家库，卖家库按卖家id分片，买家库用买家id分片，异步同步数据
  * 数据迁移问题脑子过一遍
    * [免迁移扩容](https://kefeng.wang/2018/07/22/mysql-sharding/#3-%E5%88%86%E7%89%87%E7%AD%96%E7%95%A5) 新增的库做主从复制，上线新分片规则，那些需要迁移的数据是有冗余的，择时清除即可
  * 原理  batch insert  全表group by  和事务是否有关系 
    * 主键自增问题
  * 配置表的处理 **(默认是存在第一个库的表)**
  
* rpc 

* 状态跳转的处理
 * activiti和状态机的对比
   * [activiti入门](https://juejin.im/post/5aafa3eef265da23784015b9) 跟完再对照下maf的流程
   * 跟完maf一个复杂流程，再看用squirrel怎么做 对比
 * squirrel支持导出dot文件绘制状态机跳转图 (done)

* squirrel备注问题
 * context 处理条件是否满足回调方法里调数据库
 * action 调用方法   关于方法参数的问题
 * 两个条件？？？
 * 异常处理
 * Hierarchical State **不知道解决什么问题**
 * Parallel State **不知道解决什么问题**
 * Using History States to Save and Restore the Current State
   * 能不能解决一个订单节点被打回加签之后反复走到下一个节点的展示，和数据库持久化的集成


# 特别补
* 分布式事务
* 防重，全局接口幂等是否可能？不行用分布式锁解决，怎么保证锁的可用

# 
二分模板
深优 广优 
排序 复杂度和适用情况


