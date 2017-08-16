---
title: 聊聊mysql索引
tags:
---

　　普通索引、唯一索引、全文索引、聚集索引、非聚集索引、还有什么鬼R-Tree索引……不知道各位在接触索引的时候是不是和我一样会被这一堆的名词搞的很蒙圈，其实蒙不止是概念多，而是因为索引本身是个很大很深的话题，而我们接触的东西非常的零碎。可能是mysql官网的reference、可能是网络的某些博文、可能是来自书本，而且还有一点很恐怖的，刚好所以这块**有些名词翻译成中文也没有个定准，所以特别混乱**。本文会尝试去理清这些概念和关系，算是对这些零碎的东西的总结和梳理<!--more-->

> 综上，下文的专业名词，我会写英文，英文来自mysql官方和《high performance mysql》高性能mysql这本书，应该是比较权威的，同时会给出中文的参照仅供参考。

# jingb
* jingb is a boy
* jingb is a boy
* jingb is a boy

## jingb-1
> jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jjjingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj




# eliza
> jingbsdfshkdfhsk jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jjjingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj


## djjd
jsdkhjfhksjdfhkshfk
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jjjingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj


# elizadjskfdk
> jingbsdfshkdfhsk jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jjjingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj


## djjsjdhfskjfj
jsdkhjfhksjdfhkshfk
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jjjingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj


## dskjfj
jsdkhjfhksjdfhkshfk
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj
jjjingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj

ingb jkshdkfjs  dsjdshkj
jingb jkshdkfjs  dsjdshkj


