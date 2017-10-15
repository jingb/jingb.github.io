---
title: LinuxIoModels
tags:
---

# 关键词
> 同步、异步、阻塞、非阻塞、Reactor、Proactor、select、poll……

<!-- more -->

# Synchronous I/O versus Asynchronous I/O
> * A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes
* An asynchronous I/O operation does not cause the requesting process to be blocked.
* **从发出IO操作请求开始到IO操作结束的过程中没有任何阻塞，就称为异步，否则为同步**

{% asset_img 5.png %}

# 参考资料
> * [人云思云](https://segmentfault.com/a/1190000003063859)

## two distinct phases for an input operation
> * Waiting for the data to be ready
* Copying the data from the kernel to the process

### 一个网络io的过程
> For an input operation on a socket, the first step normally involves waiting for data to arrive on the network. When the packet arrives, it is copied into a buffer within the kernel. The second step is copying this data from the kernel's buffer into our application buffer

## blocking IO
{% asset_img 1.png IO执行的两个阶段都被block了 %}

## nonblocking IO
{% asset_img 2.png 需要不断的主动询问kernel数据好了没有 %}

## IO multiplexing
{% asset_img 3.png 一调用select，那么整个进程会被block，但select函数可以监听多个socket，任何一个有数据准备就绪select就会返回 %}

### 注意
> 如果处理的连接数不是很高的话，使用select/epoll的web server不一定比使用multi-threading + blocking IO的web server性能更好，可能延迟还更大。**select/epoll的优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接**，因为单机能开的线程数是有限的，其次过多的线程间切换调度会使性能极具下降

###  There are several ways for a single thread to tell which of a set of nonblocking sockets are ready for I/O
> select
poll
epoll
/dev/poll (recommended poll replacement for Solaris)
kqueue (edge-triggered, is the recommended poll replacement for FreeBSD (and, soon, NetBSD).)

### select Fun
> ``` int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout); ```

#### 作用
> we can call select and tell the kernel to return only when
* Any of the descriptors in the set {1, 4, 5} are ready for reading(可读)
* Any of the descriptors in the set {2, 7} are ready for writing(可写)
* Any of the descriptors in the set {1, 4} have an exception condition pending(异常)
* 10.2 seconds have elapsed(超时) 

#### Under What Conditions Is a Descriptor Ready
> more specific about the conditions that cause select to return "ready" for sockets 

#### 缺陷
>  Unfortunately, select() is limited to FD_SETSIZE handles. This limit is compiled in to the standard library and user programs. (Some versions of the C library let you raise this limit at user app compile time.) 单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024，可以通过修改宏定义甚至重新编译内核的方式提升这一限制，但 是这样也会造成效率的降低

### poll
> ```int poll (struct pollfd *fds, unsigned int nfds, int timeout);```

> * 从上面看，select和poll都需要在返回后，通过遍历文件描述符fds来获取已经就绪的socket。事实上，**同时连接的大量客户端在一时刻可能只有很少的处于就绪状态**，因此随着监视的描述符数量的增长，其效率也会线性下降

### epoll
> 
* 和select、poll对比
 * select和poll的主要问题是**文件描述符数组需要在内核态和用户态之间进行复制**，并且**需要进行遍历**(大量的链接里在某一时刻只有很少处于就绪状态)才能获得就绪的文件描述符。
 * epoll则利用mmap技术避免了这些复制和遍历操作。epoll在Linux2.6出现，通过epoll_ctl进行注册，然后通过epoll_wait得到就绪通知，可以做到高效地就绪状态检查，它是目前性能最好的就绪通知方案
* 工作模式
 * level trigger(缺省的工作方式) 
 * edge trigger

## asynchronous IO
{% asset_img 4.png %}

