---
title: 编译tomcat源码
date: 2019-02-18 16:55:12
tags:
---

　　这几天把Tomcat源码下了下来想研究下，发现Tomcat现在还是用的ant做组织而不是maven，导入idea想跑源码自带的单元测试编译都过不去，只看代码不跑测试就能玩的大佬请忽略
　　都特么9102年了，还用ant的项目是真的没见过，心里千万只草泥马在跑。后来在网上找解决方案看到有人说ant和tomcat是一个作者，我特意考据了下发现是真的 
**James Duncan Davidson (born July 29, 1970 in Lubbock, Texas) is an American photographer and a software developer. While a software engineer at Sun Microsystems (1997–2001), Davidson created Tomcat, a Java-based webserver application and the Ant Java-based build tool** ——wiki百科
　　今天在tomcat官网仔细看了下，在development里有个building的简介，[链接在这](https://tomcat.apache.org/tomcat-8.5-doc/building.html)。里面有个Building with Eclipse的步骤，照着一步步搞就可以了，**亲测有效，源码自带的单元测试都能跑**。
　　**导入idea的只有这么一句，The same general approach should work for most IDEs; it has been reported to work in IntelliJ IDEA, for example.** 没有步骤，我是死活搞不定，网上查的解决方案也没有能行的，放弃了。
　　按道理说都9102年了还用个毛线的eclipse，但是这样的，idea虽然比eclipse好，但像一些用activity工作流的在eclipse里弄流程图比idea要好，你说你不用activity但MAT你总要用了吧，一时半会你也离不开它，这么想想就释然了

