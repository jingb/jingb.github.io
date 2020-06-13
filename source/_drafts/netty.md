---
title: netty
tags:
---


# ChannelInBoundHandler
一般使用ChannelInboundHandler的话，有3种方法。
* 直接实现ChannelInBoundHandler接口，一般不用
* 继承ChannelInboundHandlerAdapter，**用来处理事件状态的改变**
* 继承SimpleChannelInboundHandler，**用来处理接收消息**，这个类读取数据会自动释放资源。

**在 Netty 发送消息有两种方式。您可以直接写消息给 Channel 或写入 ChannelHandlerContext 对象。主要的区别是， 前一种方法会导致消息从 ChannelPipeline的尾部开始，而 后者导致消息从 ChannelPipeline 下一个处理器开始**

# 
ByteToMessageDecoder 
io.netty.buffer.ByteBuf （原始数据流）-> io.netty.buffer.ByteBuf （用户数据）
MessageToMessageDecoder
io.netty.buffer.ByteBuf （用户数据）-> Java Object

channelReadComplete 和 channelRead的区别  每次都writeAndFlush 还是 最后complete才flush  为了解决一个什么样的问题 [链接](https://time.geekbang.org/course/detail/100036701-171285) 

# idle实现
[dubbo的实现](https://dubbo.apache.org/zh-cn/blog/dubbo-heartbeat-design.html)
