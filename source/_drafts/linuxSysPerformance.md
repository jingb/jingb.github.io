---
title: linuxSysPerformance
tags:
---

# cgroups cpu相关
* 每个进程的 CPU Usage 只包含用户态（us 或 ni）和内核态（sy）两部分，其他的系统 CPU 开销并不包含在进程的 CPU 使用中，而 CPU Cgroup 只是对进程的 CPU 使用做了限制
* group3 中的 cpu.shares 是 1024，而 group4 中的 cpu.shares 是 3072，那么 group3:group4=1:3  cpu.shares 解决在 在一台 4 个 CPU 的机器上，当 group3 和 group4 都需要 4 个 CPU 的时候，它们实际分配到的 CPU 分别是这样的：group3 是 1 个，group4 是 3 个
* **load**和cpu使用率不是绝对相关的关系，同一台机器，把磁盘从好的换成差的，负载一样，但系统性能下降，但load体现不出来，所以后来load的计算考虑了D状态的进程

## 缺陷
D状态引起的负载升高，系统性能下载，cgroup还处理不了。因为 Cgroups 更多的是以进程为单位进行隔离，而 D 状态进程是内核中系统全局资源引入的，所以 Cgroups 影响不了它。
* 我们可以做的是，在生产环境中监控容器的宿主机节点里 D 状态的进程数量，然后对 D 状态进程数目异常的节点进行分析，比如磁盘硬件出现问题引起 D 状态进程数目增加，这时就需要更换硬盘


![参考图片](https://static001.geekbang.org/resource/image/9e/7a/9ee6c1c5d88b0468af1a3280865a6b7a.png)
# cpu相关
## load
什么含义？单位时间内，正在使用cpu和等待cpu和等待io的进程数，可以简称为活跃 -> 啥是活跃，为毛 -> ps出来 进程是R(Running或Runnable)或者不可中断(D)的，**比方磁盘写数据，不可中断，容易出现数据不一致，本质上是一种保护**，因为load看到是系统的繁忙程度，而不是cpu使用率 -> **超过70%算负载过高，比方单cpu，超过0.7**

- cpu密集型的，直接cpu跑满了；io密集型的，等待io导致负载升高；等待调度到cpu的进程非常多；3种情况可能导致负载非
- cpu使用率和平均负载不要混淆，比方io密集型的应用，等待io的进程也会使负载升高，但cpu使用率并不高。cpu密集型的，此时负载和cpu利用率的表现是一致的
- 案例是用 stress 模拟压力，用mpstat pidstat 看系统情况

看评论 
htop比较现实，忽略里面用的分析工具 
分3情况 cpu密集 io密集 大量进程的情况 **都有可能造成负载高**
负载计算的代码网上有些文章 -> 逻辑**线程**数？ 

<!-- more -->

## cpu时间的类型
{% asset_img cpu.jpeg %}

## cpu上下文切换
* 进程、线程、中断 上下文切换3种
  * 进程的切换比方进程1执行的时候某些资源条件不满足，肯定要把cpu让出来；分配的时间片用完了；自己sleep之类；高优先级的进程进来了；硬件的中断进来了，比方断电了
  * 线程的上下文切换，如果是只有一个线程，可以理解成和进程上下文切换一样；线程自己也有私有数据如栈和寄存器；多个线程共享进程的部分数据，切换的时候成本会比进程切换低一些
* **我们的监控有没有监控进程的上下文切换次数变化？没有的话是为什么？不需要业务处理还是业务处理不了？**
* vmstat cs可以查看**总体**切换次数 pidstat -w 可以看进程级别的切换情况 
* {% asset_img vmstat.png %}
r那一列是就绪队列的长度，如果比cpu核数多，则表示等待调度的进程多了
* 多长时间内多少次切换算多 和cpu性能有关系，从几百到几百万都可能是正常的，数量级的变化得关注
 * 自愿上下文切换变多了，说明进程都在等待资源，有可能发生了 I/O 等其他问题；
 * 非自愿上下文切换变多了，说明进程都在被强制调度，也就是都在争抢 CPU，说明 CPU 的确成了瓶颈；
 * 中断次数变多了，说明 CPU 被中断处理程序占用，还需要通过查看 /proc/interrupts /proc/softirqs 文件来分析具体的中断类型

## 不可中断进程和io
[案例链接](https://time.geekbang.org/column/article/71382?utm_source=pc&utm_medium=geektime&utm_content=pcchaping&utm_term=pc_interstitial_194)
**里面用了dstat去同时看cpu和io的情况，待排查vmstat能不能满足，可以的话忘记dstat**  
	试过了可以的
**案例的c代码使用了Direct Io导致io瓶颈，搞懂啥时候时候使用Direct io**
排查过程
top 负载过高 -> 发现io wait过高 -> 有D和Z状态的进程 -> vmstat 可以看到磁盘读和iowait的升高是同步的 -> pidstat -d 某个进程，发现磁盘读不多 -> ps 发现进程变成Z状态了 -> pid -d 看所有进程，确认是app进程，看他下面的所有子进程的pidstat，确实是他的问题 -> 子进程号一直变，只能用perf了，发现是用了direct io

## cpu利用率
* top看的是3秒 ps看的是进程整个生命周期
* /proc/stat /proc/$pid/stat 里cpu相关数据可以算出cpu利用率 每一列的含义可以看man proc
* {% asset_img cpu利用率计算.png %}
* perf top -p 21515 分析cpu高的进程 perf record可以持久化数据
* [cpu很高但top找不到进程的案例](https://time.geekbang.org/column/article/70822)
 * 进程不停的崩溃重启，短时进程 top监控不到 -> 因为pid一直变，无法跟踪，所以用 pstree 找这些进程的父进程 -> 找到有问题的父进程后，分析代码 -> 用perf记录一段时间的现象，验证

## 进程基础
### 状态
用ps和top有一列会展示 
运行（R）、空闲（I）、不可中断睡眠（D）Disk sleep、可中断睡眠（S）、僵尸（Z）以及暂停（T）
**不可中断状态，表示进程正在跟硬件交互，为了保护进程数据和硬件的一致性，系统不允许其他进程或中断打断这个进程。进程长时间处于不可中断状态，通常表示系统有 I/O 性能问题。**
**僵尸进程表示进程已经退出，但它的父进程还没有回收子进程占用的资源。短暂的僵尸状态我们通常不必理会，但进程长时间处于僵尸状态，就应该注意了，可能有应用程序没有正常处理子进程的退出。**

### 中断
Linux 中的中断处理程序分为上半部和下半部：
* 上半部对应硬件中断，用来快速处理中断。 也叫硬中断。
* 下半部对应软中断，用来异步处理上半部未完成的工作。
分软、硬是因为，如果中断处理程序耗时太长，可能会导致丢失中断信号

```
网卡接收到数据包后，会通过硬件中断的方式，通知内核有新的数据到了。这时，内核就应该调用中断处理程序来响应它。
对上半部来说，既然是快速处理，其实就是要把网卡的数据读到内存中，然后更新一下硬件寄存器的状态（表示数据已经读好了），最后再发送一个软中断信号，通知下半部做进一步的处理。
而下半部被软中断信号唤醒后，需要从内存中找到网络数据，再按照网络协议栈，对数据进行逐层解析和处理，直到把它送给应用程序
```

![](https://static001.geekbang.org/resource/image/59/ec/596397e1d6335d2990f70427ad4b14ec.png)
![](https://static001.geekbang.org/resource/image/7a/17/7a445960a4bc0a58a02e1bc75648aa17.png)

![](https://static001.geekbang.org/resource/image/cc/ee/ccd7a9350c270c0168bad6cc8d0b8aee.png)
![](https://static001.geekbang.org/resource/image/37/a4/37d04c213acfa650bd7467e3000356a4.png)


# 容器存储
## 静态容量限制
用了linux XFS有个quota的磁盘限制机制，docker的限制也是通过这个机制来实现的，**看看k8s怎么搞的**

## 读写性能
cgroupV1 限制不了 Buffered I/O(写到了page cache里) 只能限制 Direct I/O
{% asset_img disk.jpeg %}
因为 groupV1 在考虑io cpu 内存的时候是单独的组


# tcp
* syn flood攻击 -> client发了个syn然后下线，服务端回ack后没收到响应会重发ack，重试5次，1s + 2s + 4s+ 8s+ 16s + 32s = 63s 没成功才断开这个链接。攻击者耗尽了服务端的syn队列
* ISN（即第一个seq）的初始化，收发两端的初始值都不是1而是个随机值。和链接断开重连有关，会错乱
* 建立链接过程 ![建立链接](http://www.taohui.pub/wp-content/uploads/2017/01/accept%E9%98%9F%E5%88%97-1-1-1.jpg)
* int listen(int sockfd, int backlog) backlog在listen函数设置  sysctl net.core.somaxconn查看系统设置，应用程序也可以调整
* [backlog太小引发的应用吞吐量问题排查](https://time.geekbang.org/column/article/87342)  netstat -s 可以看有没有因队列慢导致socket丢包的

## 重传
慢启动是什么解决什么问题 -> 发送窗口初始值是2MSS -> 递增 2+2 4+4 8+8 
RTO超时重传太慢 -> 快速重传(Fast Retransmit) -> 
RTO 丢包 到 重传该包的时间 -> 发生重传的时候，拥塞窗口的调整是有讲究的，rfc定义 发了19个包，只有3个收到确认，则窗口值被定为（19-3）/2 = 8。 -> 快速重传导致的重传不需要调整拥塞窗口，因为后续的包都到了，说明网络并没有严重拥塞
快速重传面临的问题
![问题](https://coolshell.cn/wp-content/uploads/2014/05/FASTIncast021.png)
**收到3个ack2的时候不知道#3 #4 #5是不是丢了**，到底4 5 6 要不要重传
* 早期是无脑重传
* SACK(Selective Acknowledgment)解决了这个问题
* ![](https://coolshell.cn/wp-content/uploads/2014/05/tcp_sack_example-1024x577.jpg) 接收方在告诉发送方ack300的时候，把收到了哪些包都告诉发送方了，这样发送方就知道重传哪些了
* D-SACK 解决 **可以让发送方知道，是发出去的包丢了，还是回来的ACK包丢了** **网络上出现了先发的包后到的情况** 具体看[陈皓](https://coolshell.cn/articles/11564.html#SACK_%E6%96%B9%E6%B3%95) 

## rst包
这个标记用来强制断开连接，通常是之前建立的连接已经不在了、包不合法、或者实在无能为力处理 **rst包是协议栈发送的，应用程序不知道**
* 接受端控制发送端，发送端的window size为0了，这时候发送端就不发数据了。这时候tcp通过Zero Window Probe把窗口给调大，具体是发包让接收端调大他的窗口，如果3次后还是0，有些实现会直接rst包断开连接
* ![远端断电重启](https://user-gold-cdn.xitu.io/2019/6/19/16b6dd2177aff16c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

# psh
PSH（Push）：告知对方这些数据包收到以后应该马上交给上层应用，不能缓存起来

## 拥塞控制
解决加塞问题 -> 慢启动 -> 拥塞避免 -> 发生拥塞了 -> 快速恢复窗口大小


[ack seq号的变化](https://segmentfault.com/a/1190000016375111)


