---
title: LinuxIoModels
tags:
---

# 关键词
> 同步、异步、阻塞、非阻塞、Reactor、Proactor、select、poll……

# Synchronous I/O versus Asynchronous I/O
> * A synchronous I/O operation causes the requesting process to be blocked until that I/O operation completes
* An asynchronous I/O operation does not cause the requesting process to be blocked.

two distinct phases for an input operation
> * Waiting for the data to be ready
* Copying the data from the kernel to the process

> For an input operation on a socket, the first step normally involves waiting for data to arrive on the network. When the packet arrives, it is copied into a buffer within the kernel. The second step is copying this data from the kernel's buffer into our application buffer

##  There are several ways for a single thread to tell which of a set of nonblocking sockets are ready for I/O
> select
poll
/dev/poll (recommended poll replacement for Solaris)
kqueue (edge-triggered, is the recommended poll replacement for FreeBSD (and, soon, NetBSD).)

## Select Fun

### 作用
> we can call select and tell the kernel to return only when
* Any of the descriptors in the set {1, 4, 5} are ready for reading
* Any of the descriptors in the set {2, 7} are ready for writing
* Any of the descriptors in the set {1, 4} have an exception condition pending
* 10.2 seconds have elapsed 

### Under What Conditions Is a Descriptor Ready
> more specific about the conditions that cause select to return "ready" for sockets 


### 缺陷
>  Unfortunately, select() is limited to FD_SETSIZE handles. This limit is compiled in to the standard library and user programs. (Some versions of the C library let you raise this limit at user app compile time.) 
