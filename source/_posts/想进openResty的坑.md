---
title: 想进openResty的坑
tags:
  - openresty
  - nginx
  - linux
  - io
  - lua
categories:
  - linux
  - openresty
  - nginx
  - lua
date: 2017-04-20 20:44:22
---


　　最早听到openResty这个名词，可能是在某公众号里，大致意思是随着nginx的崛起，openResty和lua会<!-- more -->是后端程序猿的必备技能，当时我并不知道openResty和nginx是神马关系。日常在公司只知道公司生产环境的反向代理用的是淘宝的tengine，至于业务开发，因为没有做集群并且和运维团队隔离比较严重，基本是黑箱一个。Google过说tomcat、Apache、nginx这些服务器的区别这样类似的问题，曾经简单理解成如果请求静态资源，是可以不走后台应用服务器tomcat这种，速度会快一些。过完年回来是因为项目组的app要做小程序版，涉及到账号体系和session的处理，自己仔细看了一下项目现有的session的是怎么做的。另外自己用nginx整了一下在自己机子布多个服务，做负载均衡，并且尝试处理了如果是轮询负载的话，session的共享问题，分布式session共享的处理自己本来是有一些了解的，毕竟网上多的是现成的解决方案、博客，分布式相关的书session处理也都必定会涉及，但动手实现细节还是第一次。再加上之前在看[张开涛](http://jinnianshilongnian.iteye.com/)的shiro教程的时候发现他有一个关于openResty开发的连载，包括其他很多一些技术博客时不时总是有像nginx、lua、高性能服务器这样的标签或者关键字，慢慢地也就知道了说nginx其实不止可以做一个静态资源服务器或者简单的负载这样的功能，而openResty+lua也可以直接实现业务开发，并且在效率和性能上有卓越的提升。
　　看过的东西总是零散的，但看多了可能不知道什么时候这些东西就能联系起来。年前倒腾过一小阵子的netty，会去倒腾也无非是因为说netty是nio集大成者，那些知名的RPC框架或消息中间件，比方dubbo、activeMq……这样的中间件，底层的通讯框架清一色都是Netty。再往下看一层，为什么nginx比Apache快辣磨多，java的NIO比传统IO，也都是因为操作系统提供了IO复用的函数可以调。就是最底层最原始的网络编程那几种IO Models的理解和比较。此外，一些**协程(corutine)**的概念，可能像在go语言里支持比较好的，但在java并没有原生支持的，在lua语言里也有相应的支持。
　　包括但不限于如下几个原因，觉得这次很想入坑openResty+lua开发。
> * lua支持协程（corutine）
* nginx的IO模型适合做高性能服务器
* openresty的作者章亦春会很仔细的回复Google讨论组上的细节问题，而且几乎总是细节问题，至今一直活跃着。之前在看过一篇他的回帖，已经找不到了，有一段回复大概是“**请提供一个最小化的实例来阐述你的问题，否则这样'空对空'我没有办法和你讨论细节**”，不知道为什么总觉得类似这种回复有震撼人心的力量。而他早期写的[nginx教程](https://openresty.org/download/agentzh-nginx-tutorials-zhcn.html)(变量漫谈和配置指令执行顺序)，几乎每一个点都提供了至少一个小例子佐证或者多个例子比对，在冗长和空泛间取得了很好的平衡
* openResty在网上的几个知名推广者，都是那种学习、解决问题**先看文档**，授人以渔型那种，有什么人在学在用在推广很重要不是
* openResty现在的使用还并不广泛，但一直在发展，看得到未来
* 因为openResty+lua开发远不像java这么成熟而庞大的，很多东西都还很原生、简陋。比方好像大部分的人用的IDE都是sublime，没有成熟的比方支持断点调式这样的IDE工具。噢但自己终于有机会用vim试着做开发了哈哈，虽然我的eclipse里也装了vim的插件，但做java开发还是觉得别扭，也不是vim不熟的问题，但如果博客码字或者浏览一些lua项目和代码，竟然没有什么违和感。**一句话吧，入门门槛足够高**
* 相信会有机会和场景可以深入接触到linux的一些东西
* 可能有忘了的想起再补充……

　　基础的东西看完了总是要做点什么对吧。
> * 小米有个搞运维和安全相关的网友，用openResty做了个类似监听，**[截取数据包的东西](https://www.xsec.io/2016/7/8/nginx-lua-security.html)**，看上去炫酷的不行，当然了可能主要也是因为我们都是外行。
* 张开涛的连载教程里，最后是有两章实战教程的，其中有个[京东的商品详情](http://jinnianshilongnian.iteye.com/blog/2188538)页，不过那个示例还涉及了另外一些redis相关的中间件，没有把无关的东西剥离掉，比较复杂。作者自己在连载的前言里说过他写的这系列博客也是带组里的新人用，可能也比较多的保留了京东的真实业务场景。
* openResty有一些web相关的框架，github star最多的[lapis](https://github.com/leafo/lapis)是个老外写的没看过。比方在[openResty最佳实践](https://moonbingbing.gitbooks.io/openresty-best-practices/)里有介绍的新浪的[vanilla](https://github.com/idevz/vanilla)，也没看过哈哈……个人只看过另外一个叫[lor](https://github.com/sumory/lor)的，国货，看了一点点源码和[简介](https://orchina.org/topic/117/view)。也是搭积木嗯，比方渲染模块用的是[lua-resty-template](https://github.com/bungle/lua-resty-template)，session的处理用的是[lua-resty-session](https://github.com/bungle/lua-resty-session)……
* 火丁笔记做的[服务器推](https://huoding.com/2012/09/28/174)实现
* 至于像大的应用场景比方CDN，WAF（Web Application Firewall）水平有限都不懂嗯……openResty的作者在[官网](http://openresty.org/)也有维护一个列表，会挂一些国内国外的文章上去

　　其实本来看了一些lor的东西是打算做一个web网站练手的，但后来还是觉得做个网站并不具备**通用性**，挪了个项目就用不了了。当时就想着说要不还是做点足够通用的东西，比方权限了什么的。随便搜了个shiro和openResty的关键字可以看到网上有人用go语言+openResty仿shiro<域，动作，实体>(比方 权限<打印机，使用动作，打印机3号>)那种想法做的权限系统。自己打算用纯lua+openResty做一个权限系统的大实例，比较全面地柔和一些ngx的API和第三方库，像访问mysql、redis什么的，尽量做到通用以后也可以使用，已经做了一点点，后续做完会放到github上。
　　书的话参考的是"openResty最佳实践"和"Nginx Http Server Second Edition"，更多的还是上google的讨论组，github、openResty官网和nginx官网……**当然了，github的[awesomeXXX](https://github.com/bungle/awesome-resty)系列一如既往的详尽**。
　　以上是自己对近期学习openResty的一些概览和记录，无组织无主题，比较凌乱，后续可能会更。
