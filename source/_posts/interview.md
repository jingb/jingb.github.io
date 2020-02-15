---
title: interview
date: 2017-10-18 17:13:10
tags:
---


# 基础

## 序列化
子类、父类 设计时考虑什么问题 
Serialization proxy pattern 来自effective java 解决什么问题
serialVersionUID

## 线程和进程对比
> * [leetcode](https://discuss.leetcode.com/topic/90877/process-vs-thread/2)
* [表格](http://www.differencebetween.info/difference-between-process-and-thread)
 * 线程之间共享内存，进程之间独立内存空间
 * 线程可以由代码调度，进程由操作系统调度

<!-- more -->

## 多线程
> Java里面进行多线程通信的主要方式就是**共享内存(对象)**的方式，共享内存主要的关注点有两个：可见性和有序性。加上复合操作的原子性，可以认为Java的线程安全性问题主要关注点有3个：**可见性、有序性和原子性。**
**Java内存模型（JMM）解决了可见性和有序性的问题，而锁解决了原子性的问题。**

### 经典场景
#### 哲学家就餐问题
> * 解决方式：**资源分级**，把筷子从0到4编号，每个哲学家左手筷子的编号必须要比右手筷子小，拿筷子的时候先用左手。当四位哲学家同时拿起他们手边编号较低的餐叉时，只有编号最高的餐叉留在桌上，从而第五位哲学家就不能使用任何一只餐叉了

#### 读者写者问题
> 
* 问题：有读者和写者两组并发线程，共享一个文件
 * 允许多个读者可以同时对文件执行读操作
 * 只允许一个写者往文件中写信息
 * 任一写者在完成写操作之前不允许其他读者或写者工作
 * 写者执行写操作前，应让已有的读者和写者全部退出
* 通过信号量来解决，写者优先

### 线程
#### 线程状态
> * [文章](https://fangjian0423.github.io/2016/06/04/java-thread-state/)
* java.lang.Thread.State 枚举类
* NEW
 * Thread t = new Thread()
* RUNNABLE
* BLOCKED
 * synchronized
* WAITING
 * 执行了wait方法没带时间参数、join方法没带时间参数或者LockSupport.pack方法
* TIMED_WAITING
 * sleep、wait、join、LockSupport的parkNanos方法、LockSupport的parkUntil方法
* TERMINATED


### 队列

#### 阻塞队列
##### 代码实现
> * 用juc包里的类实现
* 数组 + notify、wait方法
* [实现](http://tutorials.jenkov.com/java-concurrency/blocking-queues.html)
以enqueue发放举例需要注意 **(1)synchronized关键字 (2)调用wait的条件 (3)队列为空时，说明之前可能有其他线程堵塞在dequeue方法，要唤醒他们**
* 会不会被别的程序调了notify  BlockingQueue实例的notify方法
   * synchronized关键字 锁类内部的 queue成员变量，不要锁BlockingQueue的实例

```plain
   public class BlockingQueue {

     private Queue queue = new LinkedList();
     private int limit = 10;
   
     public BlockingQueue(int limit) {
       this.limit = limit;
     }
     
     
     public synchronized void enqueue(Object item) throws InterruptedException  {
       while (this.queue.size() == this.limit) {
         wait();
       }
       if (this.queue.size() == 0) {
         //**唤醒那些堵塞在dequeue方法的线程**
         notifyAll();
       }
       this.queue.add(item);
     }
   
   
     public synchronized Object dequeue() throws InterruptedException{
       while(this.queue.size() == 0){
         wait();
       }
       if(this.queue.size() == this.limit){
         notifyAll();
       }
   
       return this.queue.remove(0);
     }
   
   }
```
    




##### JDK提供
> 
* ArrayBlockingQueue(bounded)
 * 底层是数组，一定要先指定队列大小
 * 构造函数有个boolean参数指定是否是公平队列，即FIFO，**默认是非公平队列**，公平队列内部是用公平的ReentrantLock实现
* LinkedBlockingQueue(bounded)
* PriorityBlockingQueue(无界，通过Comparator来实现定制型优先级排序，默认是自然序)
* DelayQueue(无界)
 * 元素实现Delayed接口
* SynchronousQueue(一个不存储元素的阻塞队列。每一个put操作必须等待一个take操作，否则不能继续添加元素)
 * SynchronousQueue的一个使用场景是在线程池里。Executors.newCachedThreadPool()就使用了SynchronousQueue，这个线程池根据需要（新任务到来时）创建新的线程，如果有空闲线程则会重复使用，线程空闲了60秒后会被回收 * 它的需求场景应该是这样的：消费者没拿走当前的产品，生产者是不能再给产品的。应该是为了保证消费者和生产者的节奏一致吧

#### 非阻塞队列
> * ConcurrentLinkedQueue(无界)
   * 基于CAS实现lock-free的队列，并发性更高


## 线程池 
> * [并发编程网链接](http://ifeve.com/java-threadpool/)
* [江南白衣](http://calvin1978.blogcn.com/articles/java-threadpool.html)

### 参数
    ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
    TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, 
    RejectedExecutionHandler handler)

#### workQueue 工作队列

#### RejectedExecutionHandler
> * AbortPolicy(默认)
* CallerRunsPolicy，A handler for rejected tasks that runs the rejected task directly in the calling thread of the execute method
* DiscardOldestPolicy
* DiscardPolicy

#### 处理流程
![线程池处理流程](http://ifeve.com/wp-content/uploads/2012/12/Java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%BB%E8%A6%81%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.jpg)

**核心线程池(corePoolSize)-->队列**-->扩容(maximumPoolSize)-->拒绝策略

#### 配置线程池考虑什么
> * 任务类型，IO还是CPU，还是混合型
* 任务的依赖性：是否依赖其他系统资源，如数据库连接
* CPU密集型任务配置尽可能少的线程数量，如配置Ncpu+1个线程的线程池。IO密集型，N核服务器，通过执行业务的单线程分析出cpu计算时间为ct，等待IO时间为io，则工作线程数（线程池线程数）设置为N\*(ct+io)/ct　　**ct、io均为占比，即一个请求里cpu占用时间和io占用时间占总这个请求总响应时间的占比**
* 混合型看能不能拆分
* 需要优先级的用PriorityBlockingQueue队列
* 要用有界的队列，不然有可能会因为**外部或内部**因素撑爆系统

### 线程中断
> * **Java中断机制是一种协作机制，也就是说通过中断并不能直接终止另一个线程，而需要被中断的线程自己处理中断。**
* Thread.isInterrupted() 测试线程是否已经中断。线程的中断状态不受该方法的影响
* Thread.interrupt() 中断线程
* **interrupt方法是唯一能将中断状态设置为true的方法。静态方法interrupted会将当前线程的中断状态清除，但这个方法的命名极不直观，很容易造成误解，需要特别注意**
* **当处理完InterruptedException后想要传播中断状态，必须要么重新抛出捕获的InterruptedException，要么通过Thread.currentThread().interrupt()重新设置中断状态**。

#### 一个实例
##### 需求
　　参考自**[文章](https://praveer09.github.io/technology/2015/12/06/understanding-thread-interruption-in-java/)**的部分代码，后面讲线程中断状态的还需再细化。
　　开个线程在控制台打印0-9九个数字，一秒打一个。线程start后，主线程休眠3秒，然后请求子线程终止，在主线程完全关闭应用之前，至多等待子线程1秒去终止。子线程需马上响应主线程的终止请求。
##### 代码
```plain
public static void main(String[] args) throws InterruptedException {
    Thread taskThread = new Thread(taskThatFinishesEarlyOnInterruption());
    taskThread.start();      
    Thread.sleep(3_000);     
    taskThread.interrupt();  
    taskThread.join(1_000);  
}

private static Runnable taskThatFinishesEarlyOnInterruption() {
    return () -> {
        for (int i = 0; i < 10; i++) {
            System.out.print(i);      
            try {
                Thread.sleep(1_000);  
            } catch (InterruptedException e) {
                break;                
            }
        }
    };
}
```

### 实现死锁的实例代码
```plain
public class DeadLock {

    public static void main(String[] args) {
        Object o1 = new Object();
        Object o2 = new Object();
        Tthread t1 = new Tthread(o1, o2);
        Tthread t2 = new Tthread(o2, o1);
        t1.start();
        t2.start();
        
        System.out.println("jingb");
    }
}

class Tthread extends Thread {
    
    Object t1, t2;
    
    public Tthread(Object t1, Object t2) {
        this.t1 = t1;
        this.t2 = t2;
    }
    
    public Tthread() {}
    
    @Override
    public void run() {
        synchronized (t1) {
            System.out.println("I am thread: " + Thread.currentThread().getName() 
                    + " and I have lock object: " + t1);
            try {
                TimeUnit.SECONDS.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            synchronized (t2) {
                System.out.println("I am thread: " + Thread.currentThread().getName());
            }
        }
    }
}

```

### ForkJoinPool
#### 关键类
> TODO

#### 和ThreadPool比较
> * **定位不同** ThreadPool更多是解决独立的相互无关系的请求，而ForkJoinPool是把一个大问题分而治之利用多核的优势汇总结果[stackoverflow关于TP和FJ的比较](https://stackoverflow.com/questions/9276807/whats-the-advantage-of-a-java-5-threadpoolexecutor-over-a-java-7-forkjoinpool)
* ThreadPool是多个工作线程**共享一个工作队列**，在里面取任务执行；而ForkJoinPool是每个工作线程有自己独立的工作队列(**work-stealing特性**)。对于任务多(由于共享工作队列，ThreadPool并发操作多)而执行时间短的场景，ForkJoinPool的表现会更好。
* ForkJoinPool应归类于parallel programming(规避了non-deterministic control flow问题，编程难度在于任务粒度的选择和通信)，而ThreadPool应归类于Concurrent Programming(网络爬虫，爬一个网站时间不确定，而non-deterministic control flow提高了复杂性)，二者的区别[stackoverflow问题](https://stackoverflow.com/questions/1897993/what-is-the-difference-between-concurrent-programming-and-parallel-programming)

#### ForkJoin与MapReduce
> * 环境差异，分布式 vs 单机多核：ForkJoin设计初衷针对单机多核（处理器数量很多的情况）。MapReduce一开始就明确是针对很多机器组成的集群环境的。
* 编程差异：MapReduce一般是：做较大粒度的切分，一开始就先切分好任务然后再执行，并且彼此间在最后合并之前不需要通信。这样可伸缩性更好，适合解决巨大的问题，但限制也更多。ForkJoin可以是较小粒度的切分，任务自己知道该如何切分自己，递归地切分到一组合适大小的子任务来执行，因为是一个JVM内，所以彼此间通信是很容易的，更像是传统编程方式。

#### Async模式
> ForkJoinPool 有一个 Async Mode ，效果是工作线程在处理本地任务时也使用 FIFO 顺序。这种模式下的 ForkJoinPool 更接近于是一个消息队列，而不是用来处理递归式的任务。


#### [论文 when to use parallel stream](http://gee.cs.oswego.edu/dl/html/StreamParallelGuidance.html)


## 单例
### 代码实现
#### double-check版(可用，仅不够优雅)
> * [如果不加volatile，会有指令重排的问题](https://conndots.github.io/2015/08/13/singleton-bug-2-jmm/) [跳到Volatile部分](#Volatile)
* singleton = new Singleton() 不是一个原子性的操作
 1. 给 singleton 分配内存
 2. 调用 Singleton 的构造函数来初始化成员变量，形成实例
 3. 将singleton对象指向分配的内存空间（执行完这步 singleton才是非 null 了）
由于编译器存在指令重拍的问题，1-2-3或者1-3-2的顺序都是可能的，如果1-3-2，假设线程t1执行了1-3，而cpu被线程t2抢占了，线程t2调用getInstance方法直接返回了singleton，但其实此时singleton还未执行上述的2步骤，会存在问题

    public class Singleton {
        private volatile static Singleton singleton = null;
        private Singleton()  {    }
        public static Singleton getInstance()   {
            if (singleton== null)  {
                synchronized (Singleton.class) {
                    if (singleton== null)  {
                        singleton= new Singleton();
                    }
                }
            }
            return singleton;
        }
    }

#### java6推荐版(兼顾了线程安全和懒加载)
    public class Singleton {
        private static class SingletonHolder {
            private static final Singleton INSTANCE = new Singleton();
        }
        private Singleton (){}
        public static final Singleton getInstance() {
            return SingletonHolder.INSTANCE;
        }
    }

#### 枚举版
    public enum EasySingleton{
        INSTANCE;
    }

## Object类
> * 自定义类要覆写hashCode方法和equals方法，因为在Collections Framework中像Set这样的集合要判重
* 在java类集中判别两个自定义对象是否相等，首先调用的是hashCode方法，其次再调用equals方法（可以程序断点验证），只有两个方法都返回true，则两个对象才算相等。

## NIO
### 准备
> 
* Linux IO Models(侧重IO multiplexing，epoll解决的问题) -> Reactor模型(引入nginx、redis) -> Netty(Boss、Worker、EventLoop)
 * Linux IO models
   * BIO单机可开线程数有限、线程调度的开销
   * **IO multiplexing优势并不是对于单个连接能处理得更快，而是在于能处理更多的连接**
   * select的fd数限制
   * poll解决了fd数限制，存在遍历所有注册在其上channel的问题，问题是海量的连接里，只有少数几个是就绪的
   * poll的另一个问题是存在fd在内核态和用户态的复制
 * Reactor模型
   * IO复用在技术层面足够了，但在软件工程层面却是不够的
   * 编码很复杂
   * redis, nginx也采用Reactor的模型


### 概览
> 
* [IO模型](http://blog.leanote.com/post/joesay/Concurrency-Model-Part-1-IO-Concurrency)
* [Scalable IO in java----Doug Lea](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)
* [黄亿华 Reactor](https://my.oschina.net/X9UvbYkw/blog/197963)


### reactor和proactor
> 
* [reactor-pattern-and-non-blocking-io](https://www.celum.com/en/blog/technology/the-reactor-pattern-and-non-blocking-io)，用Vert.x和Tomcat做了个性能测试对比
* Reactor模式(Reactor又叫Dispatcher、Notifier)
* 解决问题: 对于应用服务器，一个主要规律就是，CPU的处理速度是要远远快于IO速度的，如果CPU为了IO操作（例如从Socket读取一段数据）而阻塞显然是不划算的。**事件驱动，或者叫回调的方式，用来完成这件事情。这种方式就是，应用业务向一个中间人注册一个回调（event handler），当IO就绪后，就这个中间人产生一个事件，并通知此handler进行处理**
   * 由一个不断等待和循环的单独进程（线程）来做这件事，它接受所有handler的注册，并负责先操作系统查询IO是否就绪，在就绪后就调用指定handler进行处理，这个角色的名字就叫做Reactor
* 已经有了IO multiplexing，可以处理海量链接，为什么还需要Reactor这样的模式。
  * **技术层面足够了，但在软件工程层面却是不够的。**
  * [陶辉关于Reactor的论述](http://taohui.pub/?p=139)
* 和Proactor对比
  * Reactor模式里，操作系统只负责通知IO就绪，具体的IO操作（例如读写）仍然是要在业务进程里阻塞的去做的，而Proactor模式则更进一步，由操作系统将IO操作执行好（例如读取，会将数据直接读到内存buffer中），而handler只负责处理自己的逻辑，真正做到了IO与程序处理异步执行。所以我们一般又说Reactor是同步IO，Proactor是异步IO
* Reactor 多线程模型
 * {% asset_img 36.jpeg %}
 * mainReactor只有一个，负责响应client的连接请求，并建立连接，它使用一个NIO Selector；subReactor可以有一个或者多个，每个subReactor都会在一个独立线程中执行，并且维护一个独立的NIO Selector。这样的好处很明显，因为subReactor也会执行一些比较耗时的IO操作，例如消息的读写，使用多个线程去执行，则更加有利于发挥CPU的运算能力，减少IO等待时间。 
 * **Netty里对应mainReactor的角色叫做“Boss”，而对应subReactor的角色叫做"Worker"，4.0之后统一叫NioEventLoop**

### 同\异步 阻塞\非阻塞
* [严肃的回答](https://www.zhihu.com/question/19732473/answer/20851256)
* 同步\异步关注**消息通信机制**，在死等还是等回调
* 阻塞和非阻塞关注的是程序在等待调用结果（消息，返回值）时的状态
* **同步的意义只是说客户端发过来的数据到达之前，我干不了其他的事情。而阻塞强调的是调用一个函数，线程会不会休眠**
* **同样是很权威的几本书，包括操作系统教程，对于异步的定义都是不同的，有人把IO多路复用归为异步，有人把它归为同步，看你从哪个角度去定义同步或者异步**
 * IO复用函数，把IO可读可写的事件丢给了selector，当事件发生的时候，selector通知你，然后你自己决定下一步执行什么操作。另一种场景，**老板说你查到了就给客户打电话通知系统升级。好，这个过程中，员工问，我查到以后做什么，这其实就是请求注册一个回调函数。老板说你查到了再干某某事，这个某某事就是真正注册的回调函数。员工做完了第一件事情之后，就可以使用第一件事情的结果（电话号码）去执行第二件事了。这个过程老板甚至都不需要关心第一件和第二件事之间的切换，老板只需要通过注册回调告诉员工就行了。这才是真正的异步模型**。
 * 如果线程A服务1，2两个客户端，在服务客户端1的时候，有个远程调用要等，不拿到结果没办法接着走下一步的动作。此时线程不阻塞，即不会说卡在等待远程调用的地方休眠了，而是去服务客户端2。如果站在服务客户端1的角度而言，不等到远程调用结果不能接着走，可以把这个定义成同步。而如果从服务客户端2的角度而言，线程本身并没有阻塞。服务端在等待1的远程调用的时候，可以同时去做其他的任务。这个角度而言，也可以将IO多路复用归类为异步模型。
* 海纳关于NIO的教程，一个客户端给服务器先发被除数，再发除数，然后服务端计算结果返回的例子。用[状态机](https://zhuanlan.zhihu.com/p/27451893)和[回调模式](https://zhuanlan.zhihu.com/p/27484203)两种方式分别实现

#### 传统BIO
一个请求来开一个线程去应答，客户端请求和服务端线程数是1:1的关系。如果请求量巨大服务会挂。
#### 伪异步IO（Netty权威指南里作者提出的一种概念）
##### 比传统BIO的优点
服务端配置线程池，请求来开的线程提交到线程池调度。对比传统型是一种改进，由于有线程池的调度和有界工作队列的存在，服务至少不会挂。
##### 局限
由于读和写的函数都是同步阻塞型的，阻塞时间取决于通信双发的IO线程的处理速度和网络IO的传输速度，致命问题是**本质上讲，我们无法保证生产环境的网络状况和对端应用程序的的速度，如果对端速度很慢，我们这边速度再快也没用，可靠性太差**

#### IO复用模型
##### Buffer抽象类
老IO的数据读和写都可以直接和流打交道，而NIO只能与缓冲区Buffer通信。
##### Channel接口
###### 简介 
> * 网络数据通过Channel流入写出，双向通信。老IO的流是单向的。
* an open connection to an entity such as a hardware device, a file, a network socket, or a program component that is capable of performing one or more distinct I/O operations, for example reading or writing

###### 分类
> * 网络读写 SelectableChannel
* 文件读写 FileChannel

##### Selector
###### 简介
Selector不断轮询注册在上面的Channel，由于JDK使用了epoll函数代替传统的select函数，所以它没有最大链接句柄1024/2048的限制。**这就意味着只需要一个线程负责Selector的轮询就可以接入大量的客户端。**

### Netty
> 
* [基础组件和api sheet](https://emacsist.github.io/2017/06/25/netty%E5%AE%9E%E6%88%98%E8%AF%BB%E4%B9%A6%E7%AC%94%E8%AE%B0/)
* [网友源码学习](https://github.com/yongshun/learn_netty_source_code)
* jim写的关于netty的[线程模型](https://www.cnblogs.com/ASPNET2008/p/7106820.html)
* [Netty耗时的业务逻辑写在哪](https://www.zhihu.com/question/35487154) [参考](https://www.cnblogs.com/stateis0/p/9062151.html) 
 * 自己的业务线程池
 * 消息队列
* ChannelHandler分inbound、outbound，在ChannelPipeline中组成链条，顺序处理业务，A ChannelHandlerContext represents an association between a ChannelHandler and a ChannelPipeline and is created whenever a ChannelHandler is added to a Channel-Pipeline
* ChannelPipeline[解决了什么问题](https://github.com/code4craft/netty-learning/blob/73fd879a7dbc159a63157a6d91891b7e74783bbe/posts/ch3-pipeline.md#sendupstream%E5%92%8Csenddownstream)
* Channel : EventLoop = n : 1
* EventLoopGroup : EventLoop = n : 1
* An EventLoop is bound to a single Thread for its lifetime
* All I/O events processed by an EventLoop are handled on its dedicated Thread
* A  common  reason  for  installing  a  single ChannelHandler in multiple ChannelPipelines is to gather statistics across multiple Channels
* 面向对象表示多对多关系则需要一个中间对象，SelectionKey。selector和selectableChannel都持有这个selectionkey集合
* Boss thread: they do main work like connect, bind and pass them for real work to worker threads. Worker thread: they do the real work.(读写消息)
* in netty3, the threading model used in previous releases guaranteed only that inbound (previously called upstream) events would be executed in the so-called i/o thread(corresponding to netty4’s eventloop). all outbound(downstream) events were handled by the calling thread, which might be the i/o thread or any other. Upstream(inbound)对应上行，接收到的消息、被动的状态改变，都属于Upstream。Downstream则对应下行，发送的消息、主动的状态改变，都属于Downstream
* netty is asynchronous and event-driven. asynchronous说的是outbound io operations，event-driven 应该对应的是io inbound operations.

{% asset_img 35.png %}
{% asset_img 34.png %}

#### ByteBuf
* [零拷贝](https://segmentfault.com/a/1190000007560884)
* [池化了Buffer资源](https://my.oschina.net/andylucc/blog/614589)
* netty in action里是byteBuf usage pattern一节
 * 内存是由堆管理 HeapBuffer
 * DIRECT BUFFERS  **This aims to avoid copying the buffer’s contents to (or from) an intermediate buffer before (or after) each invocation of a native I/O operation. The Javadoc for  ByteBuffer states  explicitly,  “The  contents  of  direct  buffers  will reside outside of the normal garbage-collected heap.”**  避免了JVM堆内存和直接内存之间数据来回复制，在一些应用场景中性能有显著的提高。
  * DirectBuffer的申请和释放相比HeapBuffer效率更低，解决问题的是PooledByteBuf，即池化
  * [回收](https://stackoverflow.com/questions/6697709/are-java-directbytebuffer-wrappers-garbage-collected) 和PhantomReference有关
 * composite buffers 两者混合形式

##### 零拷贝
[零拷贝(zero copy)技术，用于在数据读写中减少甚至完全避免不必要的CPU拷贝，减少内存带宽的占用，提高执行效率，零拷贝有几种不同的实现原理](https://mp.weixin.qq.com/s?src=11&timestamp=1573968998&ver=1979&signature=WH3eUSQfgHUKBSXJL1GtTmZW9z5cFyfjGmfs-JMcQPRxj2F02UShjG4g3DoUQHV9shlIKnizfafP0jwA0cutAYK0eeaqB-tK9P53PaYoa0wWOAIs8PJKbXcNwzCkNhD6&new=1)
* RocketMQ基于mmap + write的方式实现零拷贝：mmap() 可以将内核中缓冲区的地址与用户空间的缓冲区进行映射，实现数据共享，省去了将数据从内核缓冲区拷贝到用户缓冲区

#### OutBound
* **Outbound 事件的传播方向是 tail -> customContext -> head.**
* connect是一个outbound事件，路径如下图
![](http://img.blog.ztgreat.cn/document/netty/20190121184504.png)

#### TCP粘包问题处理
> 
* 描述 {% asset_img 37.png 服务端不知道第一条消息从哪儿结束和第二条消息从哪儿开始，这种情况其实是发生了TCP粘包 %}
{% asset_img 38.png TCP拆包 %}
* 解决 
 * DelimiterBasedFrameDecoder是基于消息边界方式进行粘包拆包处理的
 * FixedLengthFrameDecoder是基于固定长度消息进行粘包拆包处理的
 * LengthFieldBasedFrameDecoder是基于消息头指定消息长度进行粘包拆包处理
 * LineBasedFrameDecoder是基于行来进行消息粘包拆包处理的

#### Bootstrap相关

{% asset_img 10.png ServerBootStrap需要两个EventGroup %}

` A server needs two distinct sets of Channels. The first set will contain a single ServerChannel representing the server’s own listening socket, bound to a local port. The second set will contain all of the Channels that have been created to handle incoming client connections—one for each connection the  server has accepted.`

<table>
    <tr>
        <th></th>
        <th>Bootstrap</th>
        <th>ServerBootstrap</th>
    </tr>
    <tr>
        <td>Networking function</td>
        <td>Connects to a remote host and por</td>
        <td>Binds to a local port</td>
    </tr>
    <tr>
        <td>Number of EventLoopGroup</td>
        <td>1</td>
        <td>2</td>
    </tr>
</table>

> Bootstrapping a client requires only a single EventLoopGroup, but a ServerBootstrap requires two (which can be the same instance)

## 集合类
### HashMap
#### 为什么需要这种结构
> 数组访问快插入慢，链表插入快访问慢，HashMap这样的结构取得了插入、访问速度的均衡

#### 实现
> * 是个数组，但数组的每个节点是个链表
* java8中，如果某个节点的链长度超过8，会用红黑树来替代数组，处理的是由于hashCode太多冲突造成的链太长而导致的查询蜕化问题。另外，get(key)发现冲突，需要用key.equals(k)去查找对应的entry

#### [哈希表的容量一定要是2的整数次幂](http://www.cnblogs.com/peizhe123/p/5790252.html)
> * 计算key放在哪个位置时，用hash值对length取模可以确保均匀
* length为2的整数次幂的话，hashValue & (length-1)就相当于对length取模

#### 为什么要二次hash
##### 过程 
> jdk1.6 HashMap的key hash值计算时先计算hashCode(),然后进行二次hash

##### 为什么
> * 由于Key的哈希值的分布直接决定了所有数据在哈希表上的分布或者说决定了哈希冲突的可能性，因此为防止糟糕的Key的hashCode实现（例如低位都相同，只有高位不相同，与2^N-1取与后的结果都相同）
* [参考文章1](http://www.jianshu.com/p/8b372f3a195d)
* [非常detail实现的文章](http://www.jasongj.com/java/concurrenthashmap/)

#### 为什么String, Interger这样的wrapper类适合作为键
> 因为String是不可变的，也是final的，无法被继承，而且已经重写了equals()和hashCode()方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，**如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象**。自定义User类作为map的key如果成员属性变动会影响hashCode和equals方法，可能会导致在容器取值时取不到

#### load factor 
#### ConcurrentHashMap和Collections.synchronizedCollection容器区别
> * ConcurrentHashMap. It allows concurrent modification of the Map from several threads **without the need to block them**. Collections.synchronizedMap(map) creates a blocking Map which will degrade performance, albeit **ensure consistency**(if used properly). Use the second option if you need to ensure data consistency, and each thread needs to have an up-to-date view of the map. Use the first if performance is critical, and each thread only inserts data to the map, with reads happening less frequently
* [stackoverflow](https://stackoverflow.com/questions/510632/whats-the-difference-between-concurrenthashmap-and-collections-synchronizedmap)
* 采用分段锁的技术，每一段都相当于一个HashTable，只要并发的操作在不同的段，就可以并发执行
* **有些方法需要跨段，比如size()和containsValue()，它们需要锁定整个表**
* **弱一致性**, 没有全局的锁，在清除完一个segments之后，正在清理下一个segments的时候，已经清理segments可能又被加入了数据，因此clear返回的时候，ConcurrentHashMap中是可能存在数据的。因此，clear方法是弱一致的 get不加锁 
* 多分区，独立锁，不代表完全没有线程竞争，当两个线程都要修改同一个分区时，其中一个要等待。而读取是没有阻塞的

## Spring
### bean的scope
> * singleton作用域的Bean只会在每个Spring IoC容器中存在一个实例，而且其完整生命周期完全由Spring容器管理
* prototype，Spring does not manage the complete lifecycle of a prototype bean
* [代码示例](http://memorynotfound.com/spring-bean-scopes-singleton-vs-prototype/)

```palin
  ApplicationContext context …… 
  
  Coffee coffee = context.getBean(Coffee.class);
  coffee.setBrand("Java");
  System.out.println("Coffee brand: " + coffee.getBrand());
  
  coffee = context.getBean(Coffee.class);
  System.out.println("Coffee brand: " + coffee.getBrand());

  singleton两次都输出 Java
  prototype第二个输出是null
```

### bean的生命周期
> 
* [网文英](https://howtodoinjava.com/spring/spring-core/spring-bean-life-cycle/)
* InitializingBean and DisposableBean callback interfaces
* \*Aware interfaces for specific behavior
* custom init() and destroy() methods in bean configuration file
* @PostConstruct and @PreDestroy annotations

### Resource Autowired Qualifier区别
> 
@Resource是按照名称来装配注入的，这个注解属于jdk的
@Autowired是按照类型装配注入的，这个注解是属业spring的
@Autowired注解一样，@Inject可以用来自动装配属性、方法和构造器；与@Autowired不同的是，@Inject没有required属性。因此@Inject注解所标注的依赖关系必须存在，如果不存在，则会抛出异常。


### BeanFactory和ApplicationContext的区别
> * [参考文章](http://www.cnblogs.com/heyongjun1997/p/5944674.html)
* 两者都是通过xml配置文件加载bean, ApplicationContext和BeanFacotry相比, 提供了更多的扩展功能
* BeanFactory是延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；**而ApplicationContext则在初始化自身是检验，这样有利于检查所依赖属性是否注入**

### Spring Bean provide thread safety?
>　　The default scope of Spring bean is singleton, so there will be only one instance per context. That means that all the having a class level variable that any thread can update will lead to inconsistent data. Hence in default mode spring beans are **not thread-safe**. However we can change spring bean scope to request, prototype or session to achieve thread-safety **at the cost of performance**. It’s a design decision and based on the project requirements.

### AOP 
> * [原理](http://www.cnblogs.com/zs234/p/3267623.html)
* aop可以由AspectJ(静态代理)这样的框架实现，在**编译时对目标类进行增强**。而Aspect这样的框架，解决的是静态代理要覆写**被代理类所实现接口的所有方法**这样重复性的问题，性能相对来说比较好，但实现也复杂
* 动态代理的实现是运行时增强
* [跳转至动态代理](#dynamic proxy)
{% asset_img aop.png %}

### IOC [原理](http://www.cnblogs.com/zs234/p/3257750.html)
> * 处理解耦的问题。类A调用B，要A处理B的初始化问题，现在把B的初始化问题丢给spring容器，解耦A、B。

### AbstractRoutingDataSource详解
用 AbstractRoutingDataSource + ThreadLocal + aop实现读写分离
[加上事务要处理的复杂情况](http://blog.trang.space/2017/04/17/%E5%BA%94%E7%94%A8%E5%B1%82%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB%E7%9A%84%E6%94%B9%E8%BF%9B/)
**Transactional 和 从库的路由注解是互斥的   要么是AOP的优先级处理  要么自定义事务处理器** 
**[事务注解需要特别注意的地方](https://www.jianshu.com/p/ec899855f049)**

### spring mvc分发过程
{% asset_img 32.jpeg %}
{% asset_img 42.png %}
{% asset_img 32.png %}

### spring boot
> * 传统意义上，我们开始一个项目整个开发环境就要搭建一个很长的时间，期间如果加入更多的特性时还要更麻烦，spring boot把大量的常见组件都组合了起来，比如 hibernate jdbc mongodb jmx等等很多很多，**只要引入相关的组件基本上就是零配置就可以用了**。
* **比较适合微服务部署方式，不再是把一堆应用放到一个Web服务器下，重启Web服务器会影响到其他应用，而是每个应用独立使用一个Web服务器，重启动和更新都很容易**。

### spring cloud
> * 从技术架构上降低了对大型系统构建的要求，使我们以非常低的成本（技术或者硬件）搭建一套高效、分布式、容错的平台
* 是一系列框架的集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册(Eureka)、配置中心、消息总线、负载均衡、断路器(Hystrix)、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。

## CopyOnWrite容器
> 
* why 
 * **不用锁**
* 应用场景
 * 读多写少的场景
* 缺点
 * 内存占用问题，由于每次都拷贝原内容，如果容器很大则会占很多内存
 * 数据一致性问题，只能保证最终一致性，而不是实时的一致性

## JVM
{% asset_img 2.png JVM整体架构，在Sun JDK中，本地方法栈和Java栈是同一个%}
> 三大部分
* Runtime Data Area
* Class Loader
* 执行引擎(Execution Engine)以及与本地方法接口(Native Interface)
 * JVM执行Java字节码的核心

### jvm Runtime Data Area

{% asset_img 20.png 注意heap区的划分，survivor分成from和to是因为young区采用复制的GC算法 %}


#### Java Stack
> Java栈由栈帧组成，一个帧对应一个方法调用。调用方法时压入栈帧，方法返回时弹出栈帧并抛弃。Java栈的主要任务是存储方法参数、局部变量、中间运算结果，并且提供部分其它模块工作需要的数据

#### Method Area(也叫No-Heap区)
> 
* 类型信息(类名、父类名、访问修饰符……)和类的静态变量都存储在方法区中
* Permenanent Generation仅在Hot Spot中和Method Area等价，其他虚拟机IBM J9等是不存在永久代概念的
* 包含了一个Runtime Constant Pool的区域
  * 具备动态性。常量不一定是编译期才能产生，即并非预置入Class文件中常量池的内容才能进入方法区的运行时常量池。比方String的intern方法
  * [参考](http://blog.csdn.net/l243225530/article/details/49736611)
    * String.intern()是一个Native方法，它的作用是：如果字符串常量池中已经包含了一个等于此String对象的字符串，则返回代表池(运行时常量池)中这个字符串的String对象；否则，将此String对象包含的字符串添加到常量池中并且返回此String对象的引用

#### Heap
> * Java堆是被所有线程共享的一块内存区域，主要用于存放对象实例
* 对象一般是直接在Eden区分配空间，大于 -XX:PretenureSizeThreshold 的直接在Old区分配空间
* young区垃圾回收使用的是复制算法，因为young区的对象大多存活时间很短，所以每次只复制很少的对象，效率高
* Full GC的垃圾收集采用的是mark-sweep或者mark-compact算法
* Young / Old = 1 / 2       –XX:NewRatio 指定
* Eden : from : to = 8 : 1 : 1        –XX:SurvivorRatio
* JVM 每次只会使用 Eden 和其中的一块 Survivor 区域来为对象服务，所以无论什么时候，总是有一块 Survivor 区域是空闲着的，新生代实际可用的内存空间为 9/10
* **survivor分成from和to是因为young区采用**[复制的GC算法](#GC copying)
* Survivor 区的对象每熬过一次 Minor GC，就将对象的年龄 + 1，当对象的年龄达到某个值时( -XX:MaxTenuringThreshold 来设定 )
* **比较常规是，Java堆大小的初始化值和最大值（通过-Xms和-Xmx选项来指定）应该是一次Full GC之后，old代活动对象的大小的3到4倍, 然后 年轻:年老=1:3**

#### 关于Direct Memory
> 
* **不属于Runtime Data Area，也不属于虚拟机规范中定义的内存区域**
* 和NIO的Channel、Buffer有关，使用Native函数库直接分配堆外内存，然后通过一个在java堆中的DirectByteBuffer对象对这块内存进行操作。**这样在一些场景中能显著提高性能，因为避免了java堆和Native堆中来回地复制数据**
* 本机内存的分配不受java堆参数设置的大小限制，但受机子本身总内存的限制。程序启动的参数设置一般会设置堆的大小，但忽略Direct Memory的设置，而在动态扩展时容易出现内存内存异常

#### java内存分配策略
> * 局部变量、形参都是从栈中分配空间
* **栈分配空间时，如为一个即将要调用的程序模块分配数据区时，应该事先知道这个数据区的大小。也就是说虽然分配是在程序运行时进行的，但是分配的大小是确定的、不变的，而这个大小是在编译期决定的而不是运行时**
* 堆在应用程序运行时请求操作系统给自己分配内存，由于操作系统管理内存分配，所以效率较栈要低。但优点在于编译器不需要知道从堆那里分配多少存储空间，也不必知道存储的数据要在堆里停留多长的时间，有更大的灵活性
* 像多态这样的机制，所需的存储空间只有在运行时创建了对象之后才能确定，必须是堆内存的分配

### GC算法
#### 关键问题
* 什么时候发生Minor GC及其过程
 * Eden满了Minor gc，升到老年代的对象大于老年代剩余空间**Major GC**
   * 发生Minor GC之前，虚拟机会先检查Old区最大可用的连续空间是否大于young区所有对象的总空间。如果大于则进行Minor GC，如果小于则看HandlePromotionFailure设置是否允许担保失败（不允许则直接Full GC）。如果允许，那么会继续检查Old区最大可用的连续空间是否大于历次晋升到Old区对象的平均大小，如果大于则尝试Minor GC（如果尝试失败也会触发Full GC），如果小于则进行Full GC。  
 * 过程
   * Eden : S1 : S0 = 8 : 1 : 1 **解决了复制GC算法浪费1/2内存的问题，因为Eden区里的对象很多都是直接被回收只有少量会被挪到Survivor区**
   * 每次进行清理时，将Eden区和一个Survivor中仍然存活的对象拷贝到 另一个Survivor中，然后清理掉Eden和刚才的Survivor


* 让大对象进入年老代
 * 为什么：大对象出现在年轻代很可能扰乱年轻代 GC，并破坏年轻代原有的对象结构。因为尝试在年轻代分配大对象，很可能导致空间不足，为了有足够的空间容纳大对象，JVM 不得不将年轻代中的年轻对象挪到年老代。因为大对象占用空间多，**所以可能需要移动大量小的年轻对象进入年老代，又由于这些年轻对象生命周期很短又在年老代，需要用Full GC回收，这对GC相当不利**
 * 设置PetenureSizeThreshold参数让大对象直接在Old区分配内存
* 对象进入年老代的年龄
 * 对象在young区每熬过一次Minor GC年龄加1，达到MaxTenuringThreshold进入年老区，默认值是15
 * 事实上，对象实际进入年老代的年龄是虚拟机在运行时根据内存使用情况动态计算的，这个参数指定的是阈值年龄的最大值。即，**实际晋升年老代年龄等于动态计算所得的年龄与-XX:MaxTenuringThreshold中较小的那个**
* 固定大小的Java堆 VS 可伸缩的Java堆
 * -Xms和-Xmx一致即稳定堆大小，可以减少GC次数，但GC时间较可伸缩的堆会慢一些
 * XX:MinHeapFreeRatio 参数用来设置堆空间最小空闲比例，默认值是 40，XX:MaxHeapFreeRatio 参数用来设置堆空间最大空闲比例，默认值是 70　　**这两个参数用于设置伸缩大小的堆**
* 关于Stop The World
 * 对象判活使用了可达性分析，而这项分析工作必须在一个能确保一致性的快照中进行，**一致性的意思是整个分析期间整个执行系统看起来就像冻结在某个时间节点上，不能出现对象引用关系还在动态变化的情况**，所以GC的时候要暂停所以应用进程
* [一道面试题](http://icyfenix.iteye.com/blog/715301)

> * GC算法 标记-清除算法(Mark-Sweep)
   * 标记和清除过程的效率都不高
   * 标记清除之后会产生大量不连续的内存碎片，空间碎片太多可能会导致，当程序在以后的运行过程中需要分配较大对象时无法找到足够的连续内存而不得不提前触发另一次垃圾收集动作
* <span id="GC copying" />复制算法(Copying)
 * **可以规避内存碎片的问题，但缺点是内存的大小相当于原来的1/2**
 * 将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉
* 标记-压缩算法(Mark-Compact)
 {% asset_img 22.png 标记完后让所有存活对象都往一端移动，再回收被标记对象 %}
* **我们常用的垃圾回收器一般都采用分代收集算法(Generational Collection)**
 * 把Java堆分为新生代和老年代，这样就可以根据各个年代的特点采用最适当的收集算法。
 * 在新生代中，每次垃圾收集时都发现有大批对象死去，只有少量存活，那就选用复制算法，只需要付出少量存活对象的复制成本就可以完成收集。
 * 老年代中因为对象存活率高、没有额外空间对它进行分配担保，就必须使用“标记-清理”或“标记-整理”算法来进行回收

### 垃圾回收器
{% asset_img 33.jpg 收集器间有连线则可以搭配使用，所属区域表明是Old区或Young区收集器 %}

> * Serial收集器(jvm client模式下默认收集器)
   * 使用一个线程去回收
   * 新生代复制算法、老年代标记-压缩；垃圾收集的过程中会Stop The World
   * -XX:+UseSerialGC
* Parallel Collector(根据Minor GC和Full GC分成以下三种)
 * -XX:ParallelGCThreads可以指定线程数
 * ParNew GC
   * -XX:+UseParNewGC
 * **Parallel GC**(jvm server模式下默认GC收集器)
   * -XX:+UseParallelGC  
 * ParallelOldGC
* CMS Collector
 * 触发规则是检查Old区和Perm区的使用率

* [面向GC的JAVA编程](http://blog.hesey.net/2014/05/gc-oriented-java-programming.html)

### 如何判断对象是否存活
> * 引用计数法，实现简单，判定高效，**无法解决对象相互循环引用的问题**
* 可达性分析法
 * 从GC Roots开始向下搜索，搜索所走过的路径称为引用链。当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的。不可达对象
 * GC Roots包括
   * 虚拟机栈中引用的对象
   * 方法区中类静态属性实体引用的对象
   * 方法区中常量引用的对象
   * 本地方法栈中JNI引用的对象

### GC分析 命令调优
> * GC日志分析
* 调优命令
* 调优工具

#### 日志分析
> 
* [http://gceasy.io/](http://gceasy.io/)在线网站直接导入gc日志可以生成统计报表
* {% asset_img 31.png %}
* {% asset_img 21.png %}
* **如果 ending occupancy1 - starting occupancy1 = ending occupancy2 - starting occupancy2 则表明这次GC对象100%被回收，没有对象进入Old区或者Perm区。如果等号前面的值大于等号后面的值，那么差值就是这次对象进入Old区或Perm区的大小。 而随着时间的增长，如果 ending occupancy2的值一直在增长，而且Full GC很频繁，那很可能就是发生内存泄露了**
* {% asset_img 20.png 注意heap区的划分 %}
* [参考文章](http://swcdxd.iteye.com/blog/1859858)
* 5.617: [GC 5.617: [ParNew: 43296K->7006K(47808K), 0.0136826 secs] 44992K->8702K(252608K), 0.0137904 secs] [Times: user=0.03 sys=0.00, real=0.02 secs]  
* 5.617（时间戳）: [GC（Young GC） 5.617（时间戳）: 
[ParNew（使用ParNew作为年轻代的垃圾回收期）: 43296K（年轻代垃圾回收前的大小）->7006K（年轻代垃圾回收以后的大小）(47808K)（年轻代的总大小）, 0.0136826 secs（回收时间）] 
44992K（堆区垃圾回收前的大小）->8702K（堆区垃圾回收后的大小）(252608K)（堆区总大小）, 0.0137904 secs（回收时间）] 
[Times: user=0.03（Young GC用户耗时） sys=0.00（Young GC系统耗时）, real=0.02 secs（Young GC实际耗时）]  

#### 调优、监控工具
> 
* BTrace 
 * [资料](http://www.jianshu.com/p/26f19095d396)
 * **解决问题**: BTrace本身也是可以独立运行的程序，作用是在不停止目标程序运行的前提下，通过HotSpot虚拟机的HotSwap技术动态插入原本不存在的调试打日志代码。传统情况是要加日志代码，走上线流程。

### 参数
> * [江南白衣](http://calvin1978.blogcn.com/articles/jvmoption-2.html)
* [网络文章](http://www.cnblogs.com/redcreen/archive/2011/05/04/2037057.html)

#### GC参数
> * -XX:+PrintGCDetails 
* -Xloggc:logs/gc.log
* JVM退出一般会在工作目录下产生一个日志文件，可以通过参数设定，-XX:ErrorFile=/tmp/log/hs\_error\_%p.log

#### heap相关参数
> * -Xms20M heap最小 
* -Xmx20M heap最大 
* -Xmn20M 年轻代大小(1.4or lator) （eden+ 2 survivor space)
* -XX:+HeapDumpOnOutOfMemoryError 可以在内存耗尽时记录下内存快照 -XX:HeapDumpPath可以指定文件路径

<span id="Class Loader" />
### Class Loader
> 分类
* BootStrap Class Loader 负责加载rt.jar文件中所有的Java类，即Java的核心类都是由该ClassLoader加载
* Extension Class Loader 负责加载一些扩展功能的jar包
* Application Class Loader 负责加载启动参数中指定的Classpath中的jar包及目录，通常我们自己写的Java类也是由该ClassLoader加载
* User Defined Class Loader  [解决什么问题](https://www.zhihu.com/question/46719811)

> 工作过程
* 装载　　将Java代码导入jvm中，生成Class文件
* 链接 
  * 校验：检查载入Class文件数据的正确性 
  * 准备：给类的静态变量分配存储空间 
  * 解析：将符号引用转成直接引用 
* 初始化　　对类的静态变量，静态方法和静态代码块执行初始化工作

#### 双亲委派模型(Parents Delegation Model)
> * **双亲委派模型要求除了顶层的启动类加载器之外，其余的类加载器都应当有自己的父类加载器**
* 下面的加载器能单向访问上层的类加载器，**启动类加载器中的类为系统的核心类,比如,在系统类中,提供了一个接口,并且该接口还提供了一个工厂方法用于创建该接口的实例,但是该接口的实现类在应用层中,接口和工厂方法在启动类加载器中,就会出现工厂方法无法创建由应用类加载器加载的应用实例问题** 比方java.sql.Driver java.sql.DriverManager源码用的是线程上下文加载器加载类 [缺陷 由Thread实例.getContextClassLoader做补充](https://www.jianshu.com/p/99f568df0f05)
* [双亲委派机制是为了安全而设计的](http://www.cnblogs.com/lanxuezaipiao/p/4138511.html)，如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完全这个加载请求时，子加载器才会尝试自己去加载
{% asset_img 9.png %}


## JMM(java memory model)
> 
* JMM与Java内存区域的划分是不同的概念层次，更恰当说JMM描述的是一组规则，通过这组规则控制程序中各个变量在共享数据区域和私有数据区域的访问方式，JMM是围绕原子性，有序性、可见性展开的
* [jenkov](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)
   * 从硬件内存模型和JVM内存模型的差异角度来阐述
* In order to shield the java developer from the differences between memory models across architectures，java provides its own memory model，and the JVM deals with the differences between the JMM and the underlying platform's memory model by inserting **memory barriers** at the appropriate places(JCIP) 
* JMM is specified in terms of actions，which include reads and writes to variables，locks and unlocks of monitors，and starting and joining with threads.

### as-if-serial
> 不管如何重排序（编译器与处理器为了提高并行度），（单线程）程序的结果不能被改变。这是编译器、Runtime、处理器必须遵守的语义

<span id="Volatiel"></span>
### Volatile
> [参考，来自jenkov](http://tutorials.jenkov.com/java-concurrency/volatile.html)

{% asset_img 1.png%}
> 假设**机子是双核，线程1在核1运行，而线程2在核2运行**，如果线程1读线程2的变量，是读的核1里的cache区的数据（**此值是从main memory拷贝而来**），非volatile变量不保证何时在本核的cache和main memory之间同步变量的值

#### 保证了两件事情
> * 被修饰的变量存于main memory(可见性问题)，而不是CPU cache，避免编译器优化带来的问题(比方instructions reordered，有序性问题)
* 假设线程1和线程2共享了对象O，线程1写了O对象的volatile变量X，线程2可以读到最新的值，同时**线程1在此(写X)之前对O对象其他非volatile变量的写操作比方有非volatile变量Y，也会被写回到main memory中，即线程2只要做了读X的动作，读到的Y也是最新的Y** (有点像一些数据库中间件，线程对主库做了写操作，则后续的读操作会被强制路由到主库，而不是从库，因为有可能因为主从复制延时的问题读不到最新值)

#### 适用场景
> * **对变量的写操作不依赖于当前值**，i++这种就不行

### happen-before(是个偏序关系)
> * [参考链接](http://ifeve.com/easy-happens-before/)
* The JMM defines a partial ordering called happens-before on all actions within the program. To guarantee that the thread executing action B can see the results of action A(whether or not A and B occur in different threads)，there must be a happens-before relationship between A and B.（JCIP）
* happens-before是比重排序、内存屏障这些概念更上层的东西，我们没有办法穷举在所有的CPU/计算机架构下重排序会如何发生，也没办法为这些重排序清晰的定义该在什么时候插入屏障来阻止重排序、刷新cache的顺序等，如果java语言提供屏障操作让java开发者自己决定何时插入屏障，那么并发代码将非常难写还很容易出错，而**hb规则恰好向Java开发者屏蔽了这些特定平台的底层细节，VM的开发者遵守hb规则（他们在开发某个平台上的VM时是清晰地知道在遇到某条规则时该在哪里插入内存屏障的），Java开发者也遵守hb规则（他们知道遵守了规定就能得到想要的结果），就能保证可见性**。


> * An unlock on a monitor happens-before every subsequent lock on that monitor.
   * 如果线程1解锁了monitor a，接着线程2锁定了a，那么，线程1解锁a之前的写操作都对线程2可见（线程1和线程2可以是同一个线程）
* A write to a volatile field happens-before every subsequent read of that volatile
 * 如果线程1写入了volatile变量v（这里和后续的“变量”都指的是对象的字段、类字段和数组元素），接着线程2读取了v，那么，线程1写入v及之前的写操作都对线程2可见（线程1和线程2可以是同一个线程）

## 锁
### Lock和synchronized关键字区别
> **简而言之是控制粒度的不同**
* 对于等待锁的线程，无法保证它们获取锁的顺序，可能出现线程饥饿（ReentrantLock构造方法可以指定是否公平）
* 无法传参数给synchronized块，比方线程A至多等待X秒请求锁，否则就超时
* synchronized关键字只能在一个方法内，而锁的lock和unlock方法可以在不同的方法中

### 实现 
[代码参考](https://wizardforcel.gitbooks.io/modern-java/content/ch5.html)

#### ReentrantLock
> 
* 通过两个继承了AQS的内部类实现公平\抢占锁和可重入的功能
  * AQS state为0表示无线程占用锁，而可重入的功能往state进行累加
* 抢占式锁的lock代码 **先直接用CAS操作去抢锁，不成功再调acquire**，公平锁是直接调用acquire(1)
```plain
final void lock() {
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    else
        acquire(1);
}
```
  * 内部类FairSync实现的tryAcquire总是会先去看队列里是否有在排队的线程

#### ReadWriteLock
> * 读写锁的理念是，只要没有任何线程写入变量，并发读取可变变量通常是安全的。所以**读锁可以同时被多个线程持有，只要没有线程持有写锁**。
* 如果读锁先被T1获取了，则T2要获取写锁也要等
* 如果写锁先被T1获取了，同时T2，T3要获取读锁，则T1释放写锁之后T2，T3可以同时获取读锁

#### StampedLock（和optimistic lock有关）

### 和Semaphore对比
> 信号量可以维护整体的准入许可。这在一些不同场景下，例如你需要限制你程序某个部分的并发访问总数时非常实用

<span id="spinLock"/>
### 自旋锁(spinLock)
#### 为什么需要自旋锁
> 
一个线程A在获得普通锁后，如果再有线程B试图获取锁，那么这个线程B将会挂起（阻塞）；试想下，如果两个线程资源竞争不是特别激烈，而处理器阻塞一个线程引起的线程上下文的切换的代价高于等待资源的代价的时候（锁的已保持者保持锁时间比较短），那么线程B可以不放弃CPU时间片，而是在“原地”忙等，直到锁的持有者释放了该锁，这就是自旋锁的原理，可见自旋锁是一种非阻塞锁。

#### 自旋锁的问题
* 如果自旋锁被持有的时间过长， 其它尝试获取自旋锁的线程会一直轮训自旋锁的状态， 这将非常浪费CPU的执行时间
* 单CPU系统上使用自旋锁，一个线程持有了便持续**占用CPU**，其他线程没有机会获得锁

#### 操作系统层面Hybird SpinLocks，Hybrid Mutexes
[并发编程网文章](http://ifeve.com/practice-of-using-spinlock-instead-of-mutex/)

### AbstractQueuedSynchronizer(关键类)
> 
* 参考
  * [在几个工具类的实现分析](http://ifeve.com/abstractqueuedsynchronizer-use/)
* **AQS为我们定义好顶级逻辑的骨架，并提取出公用的线程入队列/出队列，阻塞/唤醒等一系列复杂逻辑的实现，将部分简单的可由使用者决定的操作逻辑延迟到子类中去实现即可**
* AQS的state，这是一个由子类决定含义的“状态”。对于ReentrantLock来说，state是线程获取锁的次数；对于CountDownLatch来说，则表示计数值的大小
* 由几个关键方法去实现锁是共享\互斥，它的所有子类中，要么实现并使用了它独占功能的API，要么使用了共享锁的功能，而不会同时使用两套API 
  * protected boolean tryAcquire(int arg) 
  * protected boolean tryRelease(int arg) 
  * protected int tryAcquireShared(int arg)
  * protected boolean tryReleaseShared(int arg) 
  * compareAndSetState 成功即抢到锁


### 分布式锁
<span id="distributed lock redis" />
#### 基于redis的实现
> * [并发编程网的参考](http://ifeve.com/redis-lock/)
* [魏子珺的分析](http://weizijun.cn/2016/03/17/%E8%81%8A%E4%B8%80%E8%81%8A%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E7%9A%84%E8%AE%BE%E8%AE%A1/)

#### 关于数据库、zookeeper和redis实现分布式锁的对比
> 
* 数据库实现是把对锁的竞争丢给了数据库，存在问题包括比方服务挂了对锁的释放要在应用层用定时任务实现，要实现可重入的功能比较复杂，数据库宕机问题，最后是性能问题
* redis实现最大优势是性能，**利用过期时间解决服务执行过程中宕机而没有释放锁的问题**
* zookeeper是利用ZNode节点的临时和递增属性解决抢锁问题，用watch机制处理**服务执行过程中宕机而没有释放锁的问题**，**所以在这个问题的处理上比redis利用过期时间去释放要快很多**

##### redis基于主从切换的方案存在问题
> * redis是MS结构，MS间的数据同步是异步的
* clientA获取了M的锁，而在M将数据同步到S之前M宕机了
* S变成了M
* 由于S的数据和M不一致，可能存在clientB抢占到了clientA持有的锁

##### redis是单实例的正确实现方式
> * SET resource_name my_random_value NX PX 30000，my_random_value在所有获取锁的客户端里保持唯一
* 注意resource_name这个key的值，只能由抢占到它的客户端来删除。试想一种场景，clientA抢占到任务resource_name，但由于一个长时间的阻塞操作，在锁的超时时间之内任务还未完成，等到任务完成，clientA要删除持有的resource_name这个key，而此时可能这个锁被clientB占用着
* 关键的地方是my_random_value的设计，比方unix时间戳加上客户端id，确保不会出现上述的情况

##### Redlock算法处理redis布了集群的情况

#### 基于zookeeper的实现
> * [菩提树的文章](http://www.cnblogs.com/yjmyzz/p/distributed-lock-using-zookeeper.html)

<span id="CAS"></span>
## CAS
### [参考链接](http://www.cnblogs.com/549294286/p/3766717.html)
### why
> 构造无锁的数据结构，提高并发性

### 原理
### ABA问题
> 可以用java AtomicStampedReference来处理ABA问题

### 和optimistic lock关系

## servlet
> 
* [工作原理](https://stackoverflow.com/questions/3106452/how-do-servlets-work-instantiation-sessions-shared-variables-and-multithreadi)
 * 生命周期，init一次
 * 是否是单例
 * 是否是线程安全
 * 是单例为什么可以同时处理多个请求
* [单例](https://stackoverflow.com/questions/8011138/servlet-seems-to-handle-multiple-concurrent-browser-requests-synchronously)

异步servlet
* [网络连接依旧在](https://blog.csdn.net/zhurhyme/article/details/76228836)

## 设计模式
* 适配器
 * 系统中有旧类可以实现功能，但客户端的接口和旧类的接口不匹配，需要一个中间链接角色来适配，达到复用旧类的目的
 * {% asset_img 30.png %}
 * 实现一个接口类不想实现其所有方法，可以继承其抽象子类(适配器类\*\*adapter)只覆写想要覆写的方法
 * [插线口标准实例](http://blog.csdn.net/zhangjg_blog/article/details/18735243)
 ```plain
   public class SocketAdapter implements GermanSocketInterface {   //实现旧接口  
       private ChineseSocketInterface gbSocket;  
       public SocketAdapter(GBSocketInterface gbSocket) {  
           this.gbSocket = gbSocket;  
       }  
       /** 
        * 将对旧接口的调用适配到新接口 
        */  
       @Override  
       public void powerWithTwoRound() {  
           gbSocket.powerWithThreeFlat();  
       }  
   }  
 ```
* 工厂
* 装饰

## linux and JVM commands 
### 系统类相关
#### top
#### iostat
> * [参考1](http://www.ha97.com/4546.html)
* [参考2](http://www.cnblogs.com/peida/archive/2012/12/28/2837345.html)
* [参考3](http://linuxperf.com/?p=156)
* 如果%iowait的值过高，表示硬盘存在I/O瓶颈，%idle值高，表示CPU较空闲，如果%idle值高但系统响应慢时，有可能是CPU等待分配内存，此时应加大内存容量。%idle值如果持续低于10，那么系统的CPU处理能力相对较低，表明系统中最需要解决的资源是CPU
* -x参数输出磁盘extended信息，比方%util 如果%util接近100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈。如果 svctm 比较接近 await，说明 I/O 几乎没有等待时间；如果 await 远大于 svctm，说明I/O 队列太长，io响应太慢，则需要进行必要优化。如果avgqu-sz比较大，也表示有当量io在等待

#### free(mac用vm_stat)
> * [参考1](http://www.cnblogs.com/coldplayerest/archive/2010/02/20/1669949.html)
* [参考2](http://www.php-oa.com/2008/04/04/linux-free.html)

#### vmstat(mac用vm_stat)
> * [英文资料有示例](https://www.thomas-krenn.com/en/wiki/Linux_Performance_Measurements_using_vmstat#CPU_User_Load_Example)
   * cpu intensive 
    * CPU User Load 用户进程执行cpu密集的任务，比方A standard audio file will be encode as an MP3 file，cpu相关列的us很高
    * CPU System Load 比方dd if=/dev/urandom of=500MBfile bs=1M count=500这个指令，生成随机数往文件里写，cpu相关列的sy很高
   * 缺内存
    * RAM Bottleneck (swapping) 内存不足，so(swap out - memory swapped to disk)会很高
   * IO intensive
    * High IO Read Load  A large file (such as an ISO file) will be read and written to /dev/null using dd
    * High IO Write Load  dd will read from /dev/zero and write a file. 

   * CPU Waiting for IO (见参考) 
* [实际机子情况分析](https://serverfault.com/questions/54546/how-would-you-interpret-the-following-vmstat-output)
* [参考](http://www.cnblogs.com/ggjucheng/archive/2012/01/05/2312625.html) 
* vmstat看swpd大于0且si so数值较大说明内存不够用，考虑加内存条，top进入监控后，可以按M按占用内存多少进行排序展示，P按cpu

#### strace(mac用dtruss)

### JVM相关
> 
* [李艳鹏文章](http://www.jianshu.com/p/46a120f9e5a3)
* [纯洁的微笑](http://www.ityouknow.com/java/2016/01/01/jvm%E8%B0%83%E4%BC%98-%E5%91%BD%E4%BB%A4%E7%AF%87.html)

#### jstat
> 
* 主要看GC情况，与jmap对比，jstat更倾向于输出积累的信息与打印GC等的统计信息等。 
* jstat -gccapacity 1262 2000 20  监控pid为1262的程序每2秒输出一次输出20次
* -gcutil可以输出占用百分比

#### jmap
> * jmap -dump:live,format=b,file=dump.hprof 28920
* jmap -heap 28920      打印heap的概要信息，GC使用的算法，heap的配置及wise heap的使用情况

#### jhat和MAT
> 
* 分析用jmap dump下来的hprof文件

#### jstack
> 
* 生成虚拟机当前时刻的线程快照，帮助定位线程出现长时间停顿的原因，分析查看线程的状态、优先级、描述等具体信息
* 死锁检测
* [参考](http://www.hollischuang.com/archives/110)

#### jinfo
> 
jinfo可以输出并修改运行时的java进程的环境变量和虚拟机参数。

# 熟悉

## TCP

### 参考
> 
* [就是要你懂TCP](http://jm.taobao.org/2017/06/08/20170608/) 蛰剑关于TCP的文章很优质
* [陶辉关于TCP连接和消息发送](http://taohui.pub/?p=68) 半连接和全连接队列
* [TCP简介-阮一峰](http://www.ruanyifeng.com/blog/2017/06/tcp-protocol.html)
* [网文](https://github.com/jawil/blog/issues/14)

### 引入
> 以太网解决问题 -> IP层解决的问题 -> TCP要解决可靠传输的问题 -> 3次握手(子问题为什么是3次)

### 作用
> 
* 以太网解决的是局域网内点对点的通信问题
* IP解决的是局域网间的通信问题
* IP 协议只是一个地址协议，并不保证数据包的完整，TCP 协议的作用是，保证数据通信的完整性和可靠性。**tcp是可以可靠传输协议，它的所有特点都为这个可靠传输服务**
  * 丢包（网络分组在传输中存在的丢失）
  * 重复（协议层异常引发的多个相同网络分组）
  * 延迟（很久后网络分组才到达目的地）
* ack总是seq+len（包的大小），这样发送方明确知道server收到那些东西了。但是特例是三次握手和四次挥手，虽然len都是0，但是syn和fin都要占用一个seq号，所以这里的ack都是seq+1。

### 几点保证可靠
* 丢包，ack回复解决
* 服务端一段时间没收到ack，会重发
* 滑动窗口解决 传输数据每次都要等 seq + ack 的问题，批量回复ack

### 概念
> 
* MTU
 * **数据链路层**，都会对网络分组的长度有一个限制。例如以太网限制为1500字节，802.3限制为1492字节。当内核的IP网络层试图发送报文时，若一个报文的长度大于MTU限制，就会被分成若干个小于MTU的报文，每个报文都会有独立的IP头部
 * **这种IP层的分片效率是很差的**，因为必须所有分片都到达才能重组成一个包，其中任何一个分片丢失了，都必须重发所有分片。所以，TCP层会试图避免IP层执行数据报分片
* <span id="MSS" />MSS 
 * **为了避免IP层的分片**
 * 定义了一个TCP连接上，一个主机期望对端主机发送单个报文的最大长度。TCP3次握手建立连接时，连接双方都要互相告知自己期望接收到的MSS大小
* Sequence Number是包的序号，用来解决网络包乱序（reordering）问题
* Acknowledgement Number就是ACK——用于确认收到，用来解决不丢包的问题
* {% asset_img 40.jpg %}

### 三次握手
> 
* 握手的目的
 * {% asset_img 39.png  %}
 * **告知和协商一些信息**
   * [MSS–最大传输包](#MSS)
   * SACK_PERM–是否支持Selective ack(**优化重传效率**）
   * WS–窗口计算指数
 * 为什么是3次握手
   * 这个问题的本质是, 信道不可靠, 但是通信双发需要就某个问题达成一致. 而要解决这个问题, 无论你在消息中包含什么信息, 三次通信是理论上的最小值. 所以三次握手不是TCP本身的要求，**而是为了满足"在不可靠信道上可靠地传输信息"这一需求所导致的**

### 四次挥手
> 
* 三次握手的第二步发syn+ack，如果拆分成两步先发ack再发syn完全也是可以的（效率略低）**之所以绝大数时候我们看到的都是四次挥手，是因为收到fin后，知道对方要关闭了，然后OS通知应用层要关闭啥的，这里应用层可能需要做些准备工作，有一些延时，所以先回ack，准备好了再发fin 。 握手过程没有这个准备过程所以可以立即发送syn+ack。**
* 主动断连方为什么要这有TIME_WAIT不直接给转成CLOSED状态
  * TIME_WAIT确保有足够的时间让对端收到了ACK，如果被动关闭的那方没有收到Ack，就会触发被动端重发Fin，一来一去正好2个MSL

### TIME_WAIT数量过多解决
> * {% asset_img 15.png 主动发起断链的会进入TIME_WAIT状态 %}
* 设置tcp_tw_reuse和tcp_tw_recycle参数，HTTP服务器设置keep-alive参数，让客户端断链
* **使用tcp_tw_reuse和tcp_tw_recycle来解决TIME_WAIT的问题是非常非常危险的，因为这两个参数违反了TCP协议**
* 许令波的深入分析java web技术内幕关于tcp参数调优
* [丁火笔记](https://huoding.com/2012/01/19/142)，TIME_WAIT网络故障记录
* [丁火笔记](https://huoding.com/2013/12/31/316)
* [酷壳](http://coolshell.cn/articles/11564.html)
* [王宏江](http://hongjiang.info/nginx-tomcat-keep-alive/)

### TCP滑动窗口
> 
* 服务器发送数据包，当然越快越好，最好一次性全发出去。但是，发得太快，就有可能丢包。带宽小、路由器过热、缓存溢出等许多因素都会导致丢包。线路不好的话，发得越快，丢得越多
  * 最理想的状态是，在线路允许的情况下，达到最高速率。但是我们怎么知道，对方线路的理想速率是多少呢？答案就是慢慢试
* TCP必需要知道网络实际的数据处理带宽或是数据处理速度，这样才不会引起网络拥塞，导致丢包

### 其他
> 
* Nagle算法是用来优化改进tcp传输效率的

## HTTP
### 长链接
> 
* HTTP 1.0+ 用Keep-Alive
* HTTP 1.1 用persistent

### 缓存
　　当我们在一个项目上做http缓存的应用时，我们还是会把上述提及的大多数首部字段均使用上，例如使用 Expires 来兼容旧的浏览器，使用 Cache-Control 来更精准地利用缓存，然后开启 ETag 跟 Last-Modified 功能进一步复用缓存减少流量。

### 协议首部字段
> * 通用首部(Date、Cache-Control、编码……)，请求首部，响应首部

## redis
### 数据类型
{% asset_img 6.jpg %}

> [文章](http://www.cnblogs.com/qunshu/p/3196972.html)

#### Hash
　　像memcached不支持hash结构，只能在应用层每次序列化和反序列化，不仅麻烦，而且对于一些并发的场景要另外加锁。
#### Set
　　利用Redis提供的Sets数据结构，可以存储一些集合性的数据，比如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有粉丝存在一个集合。Redis还为集合提供了求交集、并集、差集等操作，可以非常方便的实现如共同关注、共同喜好、二度好友等功能，对上面的所有集合操作，你还可以使用不同的命令选择将结果返回给客户端还是存集到一个新的集合中。

### Redis的并发竞争问题如何解决
**两种思路**
> 
* 数据库层面加锁
```plain
  WATCH mykey
  val = GET mykey
  val = val + 1
  MULTI
  SET mykey $val
  EXEC
```
* 应用层层面加锁，**会有问题，类似RRD客户收益汇总**
```plain
  synchronized void businessMethod() {
    val = get mykey
    val = val + 1
    set mykey val 
  }
```




### Memcached和Redis的区别
> * 数据类型，Memcached只支持字符串
* 线程模型
* 持久化的支持
* Memcached本身并不支持分布式，因此只能在客户端通过像一致性哈希这样的分布式算法来实现Memcached的分布式存储

### 单线程模型
#### 单线程-多路复用IO模型
#### 为什么是单线程模型
> * redis和传统的多线程服务器不同。比如tomcat这些，后端往往存在很重的IO操作，会产生长时间的等待。所以采用多线程是有必要的。
* Redis基本是内存操作，在IO和网络操作的时候，多线程的程序可以很好的利用CPU时间。那在基本是内存操作的情况下，单线程程序应该可以充分利用CPU时间了

#### 存在问题
当进行一些复杂的集合操作的时候会使redis并发性下降 
可以在一个多核的机器上部署多个redis实例。组成master-master，master-slave的形式，实现读写分离。耗时的读命令完全可以放到slave中

### 实现分布式锁
[跳转](#distributed lock redis)

### 集群
> [知乎链接](https://www.zhihu.com/question/21419897)
* 客户端分片
 * jedis支持客户端分片，甚至有keyTagPattern这样的模式，即抽取key的一部分keyTag做sharding，这样通过合理命名key，可以将一组相关联的key放入同一个Redis节点，这在避免跨节点访问相关数据时很重要
* 代理分片
 * Twemproxy，最大的痛点在于，无法平滑地扩容/缩容
 * codis
* Redis Cluster
 * 请求发到任意一个redis实例，由redis cluster判断这个请求应该到哪个实例，再回给客户端重新进行请求。类似重定向

### Sentinel保证高可用
> * [庄周梦蝶的文章](http://blog.fnil.net/blog/255ccfae971d6d30b0921120d327490b/)
* [redis官方教程](http://redis.majunwei.com/topics/sentinel.html)

```plain
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```
> 
* 设置了一个实例叫mymaster，另一个叫resque
* sentinel monitor <master-group-name> <ip> <port> <quorum>  
假设有5个sentinel进程，quorum在这里是2，则当有2个sentinel发现mymaster挂了的时候，其中一个会尝试发起故障转移的动作，只要当至少3个(5 / 2 + 1)sentinel认定mymaster挂了的时候，故障转移流程会真正开始

## 异步servlet

# 了解

## RPC
> 
* 为什么需要
  * 用**动态代理**实现透明化远程服务调用(封装通信细节让用户像以本地调用方式调用远程服务)
  * 用nginx这样的软负载也可以做到负载均衡服务自动注册发现下线通知，甚至扩容的时候不用重启nginx或者reload nginx.conf文件。那为什么还需要RPC？做**服务治理功能，比方熔断、限流、服务版本等问题**
* 实现功能
  * 对消息进行编码和解码
  * 通信框架
  * 发布自己的服务
   * 人肉
   * 自动，zookeeper，leader选举机制可以确保服务的高可用
* 服务调用超时问题怎么解决
  * dubbo在调用服务不成功时，默认是会重试两次的，对于核心的服务中心非幂等写服务，去除dubbo超时重试机制，并重新评估设置超时时间
  * 幂等设计
  * 提前生成唯一流水号来保证写操作通过流水号实现幂等操作，提供查询接口

## dubbo

* 单一长连接怎么做并发
[网文](https://blog.kazaff.me/2014/09/20/dubbo%E5%8D%8F%E8%AE%AE%E4%B8%8B%E7%9A%84%E5%8D%95%E4%B8%80%E9%95%BF%E8%BF%9E%E6%8E%A5%E4%B8%8E%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%B9%B6%E5%8F%91%E5%A6%82%E4%BD%95%E5%8D%8F%E5%90%8C%E5%B7%A5%E4%BD%9C/)

* 超时设置的优先级
  客户端方法级>客户端接口级>客户端全局>服务端方法级>服务端接口级>服务端全局
* [dubbo 明细](/2018/11/17/dubbo/) 


## RxJava

## java8
> * stream
* lamda
* CompletableFuture

## 缓存

## 消息中间件

### 什么时候用
[沈剑的文章](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651960012&idx=1&sn=c6af5c79ecead98daa4d742e5ad20ce5&chksm=bd2d07108a5a8e0624ae6ad95001c4efe09d7ba695f2ddb672064805d771f3f84bee8123b8a6&scene=21#wechat_redirect)

### ActiveMQ
#### 集群配置方式有哪些
> 
* [基于Network Bridge的负载均衡方案](http://blog.csdn.net/yinwenjie/article/details/51124749)
* [高可用方案](http://blog.csdn.net/yinwenjie/article/details/51205822)
* MASTER/SLAVE模式
   * Pure master slave
{% asset_img 13.png %}
   * 基于共享文件系统(KahaDB)
{% asset_img 14.png %}
   * 基于数据库
   * 5.9版本后推荐基于levelDB和zookeeper
* Cluster模式
 * networkConnector的**duplex属性**
 * Broker clusters（静态static）
 * network of brokers（动态multicast）
   * 原理：基于组播（multicast）的节点发现
   * [network of brokers的运作机制和可能存在的无法做到高可用的问题](http://www.cnblogs.com/yjmyzz/p/activemq-ha-using-networks-of-brokers.html)
   * brokerA和brokerB通过桥接链接起来 假设A有消息但无消费者，如果B有消费者可以消费A里的消息，但消息只存于A而不是A、B有两份，所以如果此时A宕机了会发生A中的消息不能被消费的情况直到A恢复，不保证高可用 {% asset_img 41 %}
   * [network of brokers搭建集群方案](http://www.cnblogs.com/yjmyzz/p/activemq-broker-cluster.html)

## mysql

### mysql参考资料
> 
* [何登成关于锁处理幻读问题的文章](http://hedengcheng.com/?p=771#_Toc374698307)
* [南大nyankosama关于索引的文章](http://www.nyankosama.com/2014/12/19/high-performance-index/)
* [索引设计概要的网文](https://draveness.me/sql-index-intro.html)

### 关于innodb引擎的MVCC
> 
* 是什么？ Multi-Version Concurrency Control，**读不加锁，读写不冲突**，性能好。另一种模式是基于锁的并发控制，Lock-Based Concurrency Control
* mvcc中的读分类
  * snapshot read，读取的是记录的可见版本 (有可能是历史版本)，不用加锁
    * select * from table where ?;  也有可能是current read，看where条件
  * current read，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录
    * select * from table where ? lock in share mode;
    * select * from table where ? for update;
    * insert into table values (…);
    * update table set ? where ?;
    * delete from table where ?;

### innodb锁的分类
> 
* Intention Lock
  * Intention shared, IS  SELECT … LOCK IN SHARE MODE
  * Intention exclusive, IX  SELECT … FOR UPDATE
* Shared and Exclusive Locks
* Record Locks
* Next-Key Locks
  * 行锁和间隙锁组合起来就叫Next-Key Lock
* Gap Locks
  * 锁定索引记录间隙，确保索引记录的间隙不变。
  * **是RR隔离级别相对于RC隔离级别，不会出现幻读的关键**

### 索引
#### 原理
> * 目前大部分数据库系统及文件系统都采用B-Tree或其变种B+Tree作为索引结构
* 索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗，相对于内存存取，I/O存取的消耗要高几个数量级，所以评价一个数据结构作为索引的优劣最重要的指标就是在查找过程中磁盘I/O操作次数的渐进复杂度。**换句话说，索引的结构组织要尽量减少查找过程中磁盘I/O的存取次数**
* 局部性原理与磁盘预读
 * 当一个数据被用到时，其附近的数据也通常会马上被使用
 * 磁盘往往不是严格按需读取，而是每次都会预读，即使只需要一个字节，磁盘也会从这个位置开始，顺序向后读取一定长度的数据放入内存
 * 红黑树或者二叉搜索树这种结构，由于逻辑上很近的节点（父子）物理上可能很远，无法利用局部性原理，适合实现HashMap这样在内存的数据结构

#### 建索引方式
> 
* BTree
  * 长文本字段建索引方式 
  * Index selectivity概念 
* Hash

#### 索引列的顺序选择
> 
* 经验法则是把selectivity大的列建在索引前面
* 频率最大原理设计索引
  * 婚恋类的业务，几乎每个查询用户相关资料的业务都要查询性别的字段，尽管它的selectivity非常低，但还是将性别字段建在索引的第一列更好
* 对于使用or的情况，用union all代替

#### mysql explain计划
> 
* covering index, Extra字段如果是using index则代表了这个查询是一个覆盖查询，完全使用了索引来查询数据
  * select t.v1 from test t 
  * select * from test 
  * **其中v1字段建了索引，则第一个语句explain后的extra字段是using index**

#### clustered index(聚簇索引)
> 
* 参考 
  * [mysql 5.7官方文档](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)
  * [网络文章参考](http://blog.csdn.net/ak913/article/details/8026743)
* 不是某种索引，而是**数据组织方式**，像字典的以拼音查，逻辑相邻物理位置也相邻，而以偏旁部首查则逻辑相邻不一定物理相邻。
* **由于clustered index，在逻辑上相邻(比方主键1和2)的数据在物理上也是相邻的，select * from t where t.id <2这样的语句可以利用到硬盘的预读原理减少了访问磁盘的次数，所以速度会很快**
* mysql为表建clustered索引的选择
  * 表指定的递增主键
  * 表无主键，则取第一列not null的unique索引
  * 上述两条都不符合，则InnoDB internally generates a hidden clustered index on a synthetic column containing row ID values
* **所有不是clustered index的索引都是secondary index**


### 事务级别
|                             | 脏读   |  不重复读(NonRepeatable read) |  幻读(Phantom Read)  |
| --------                    | :----: |  :----:                       | :----:  |
| 未提交读(read uncommitted)  | 存在   |   存在                        |  存在   |
| 已提交读(read committed)    | X      |   存在                        |  存在   |
| 可重复读(repeatable read)   | X      |   X                           |  存在   |
| 串行(serializable)          | X      |   X                           |    X    | 
>  
* **针对当前读，RR隔离级别保证对读取到的记录加锁 (记录锁)，同时保证对读取的范围加锁，新的满足查询条件的记录不能够插入 (间隙锁)，不存在幻读现象**
* [关于mysql innodb用锁实现事务隔离](http://blog.sina.com.cn/s/blog_499740cb0100ugs7.html)
* [参考](http://www.cnblogs.com/zhoujinyi/p/3437475.html)
* 脏读，事务A读了事务B还未提交的数据
* 不可重复读，事务A在事务过程中多次读取某个数据，而此数据可能在事务B中修改过导致A多次读取的数据不一致
* 所谓幻读，就是同一个事务，连续做两次当前读 (例如：select XX from t1 where id = 10 for update;)，那么这两次当前读返回的(**记录数量一致，记录本身也一致**)，第二次的当前读，不会比第一次返回更多的记录(幻象)
* **InnoDB 使用 next-key locks for searches and index scans, which prevents phantom rows(幻读)**


### 数据库遇到性能瓶颈的处理顺序
> 数据库水平分片作为数据库性能瓶颈的最终解决方案，确实可以有效的解决这个问题。但是它将业务逻辑变得非常复杂（主要是**关联表冗余**和**字段冗余**，以及这些数据的更新），并且有分布式事务这个难题。所以不到必要时刻，尽量不要轻易尝试数据库分片。而是先去选择对业务影响较小的读写分离，垂直分库等方案。
[见文](http://www.scienjus.com/database-sharding-review/)

### replication
> 
* 基于binary log
* 方式
  * one-way, asynchronous replication M给S写数据，一旦网络延迟或中断，则Slave和Master就会出现数据不一致；甚至如果此时Master宕机，会导致未发送到slave的数据丢失
  * semisynchronous replication
  *  同步写slave

### 高可用
> * [参考](https://www.percona.com/blog/2016/06/07/choosing-mysql-high-availability-solutions/?from=timeline&isappinstalled=0)
* [老叶茶馆](http://imysql.com/2015/09/14/solutions-of-mysql-ha.shtml)
* 基于主从复制
 * 双节点主从 + keepalived/heartbeat
 * MHA
* 基于Galera协议
 * MariaDB Galera Cluster
 * Percona XtraDB Cluster


### sharding 和 partition
> * Partitioning is a general term used to describe the act of breaking up your logical data elements into multiple entities
* Sharding is the equivalent of "horizontal partitioning"
* sharding to mean the partitioning of a table **over multiple machines** (over multiple database instances in a distributed database system), whereas partitioning may just refer to the splitting up of a table **on the same machine**

### 关于切分
#### 基于应用层或代理
> * 代理
   * 应用就像直接访问单库一样访问数据库，把分库逻辑封装在代理层，缺点是稍慢
   * 支持多种编程语言
   * 360的Atlas、阿里的Cobar、Mycat等
* 应用层
 * sharding-jdbc

#### 异构索引
> * 解决分布式场景下数据拆分维度和数据查询使用维度不一致导致的低效问题
* 当数据表被拆分为多个分库分表时，数据在分库分表的分布规则就固定了。但是通常数据的业务使用场景非常复杂，如果数据的查询维度和数据拆分分布的规则一直，单条SQL会在一个分库分表上执行；如果数据的查询使用维度和数据拆分分布的规格不一致，单条SQL可能在多个分库分表上执行，出现跨库查询，跨库查询会增加IO成本，查询效率必然下降
* 解决这个问题的思路还是分布式数据库的一贯原则，让SQL执行在单库上完成，实际采用的方式就是用“空间换效率”的方案，也就是将同一份数据表，冗余存储多份，按照不同的业务使用场景进行拆分，保持拆分维度和使用维度统一，而多份数据之间会实时数据复制以解决数据一致性问题，这就是“异构索引”方案。

#### 维度
##### 水平分表和分片(sharding)
> * 参考
   * [聚合](http://www.cnblogs.com/maanshancss/p/5327803.html)
   * [黄钧航](http://blog.csdn.net/qingrx/article/details/8756839)
  * 水平分表是在同一库在对表进行水平，字表间仍可以通过union这样的关键字进行联合查询
* [王滔博客](https://www.cnblogs.com/wangtao_20/p/7115962.html)

##### 水平分片的sharding key
> * Sharding扩容与系统采用的路由规则密切相关：基于散列的路由能均匀地分布数据，但却需要数据迁移，同时也无法避免对达到上限的节点不再写入新数据；基于增量区间的路由天然不存在数据迁移和向某一节点无上限写入数据的问题，但却存在“热点”困扰
* 选择
 * 基于id的hash，可以使用一致性hash
 * id简单分区间，比方1-2w在A库，2w-4w在B库
 * 基于时间分片
 * 基于地理位置

######  库 和 表 index的索引计算
db0 和 db1，对于某个逻辑表tb，各自分别有 tb0, tb1, tb2
比方按 用户id 做sharding key，
则库的 下标 = uid % 2，2是库的个数
表的 下标 = uid / 3 % 3，3是表的数量

###### 按uid分片，用订单号查询怎么查
* **思路：既然是根据用户id来分散订单数据的。那么只要知道了这个订单号是谁的(得到了用户id)，就能知道去哪个库、哪个表查询数据了**。建一个订单和用户的关系表order_user_idx，根据订单号先查出uid，知道单存在哪个库，再去这个库用单号查数据


##### 关联表的冗余字段问题

查询用户 1 关注的所有男性用户，并且以每页 20 条进行分页
```plain
单库情况
select * 
from user u 
inner join follow f on f.follow_id = u.user_id 
where f.user_id = 1
```

```plain
多库的情况
ids = 
select follow_id from follow where user_id = 1 limit 20;
use db_1;
select * from user where id in (ids) amd sex = 1;
use db_2;
select * from user where id in (ids) amd sex = 1;
```

**上述查询可能出现数据量不足的情况，需要把sex字段从user表冗余到follow表中去**

##### 关联表的冗余表
> **对于表之间的关联关系，尽量将一对多，多对一中的关联数据放在同一个分片下（例如一个用户和他所发的所有微博哦），多对多关系最坏情况需要在在关联双方所在的数据库都存放一条记录，并且保持这多条记录的同步更新。**

##### 水平分片的分页查询解决方法 
> 比方表t被水平拆分成表t1和t2，则取出前10条数据时，需在t1取10条，t2取10条，合并后再取最前面10条。问题是翻页越翻到后面，取第1000条到1010条数据，则一共要取2000+条数据出来排序，效率极大降低
[解决方案](http://www.10tiao.com/html/249/201702/2651959942/1.html)
* 直接t1取1010，t2取1010，在应用层合并后排序取出10条(暴力法)
* 折衷法，不让直接跳页查询。比方t1取了第一页10条数据后，记录id的最大值t1_max_id，这样t1在点下一页时，sql语句是 select * from t1 where id > t1_max_id limit 10，相比于 select * from t1 limit 20要返回两页数据，这在越翻页到后面性能相比暴力法越明显。当然同样离不开的是t2也照此方法处理，然后20条数据同样要在应用层排序


#### 解决方案
##### 代理
> * 对应用透明，工程上讲是更好的方式
* 不受限于语言

##### 应用层中间件
> * 性能更好，对应用的侵入性较强

### 读写分离
> * 默认是asynchronous同步，5.7的版本支持semisynchronous

#### 缺陷
> * The single Master server for writes is a clear limit to scalability, and can quickly create a bottleneck
* The Master/Slave replication mechanism is “near-real-time,” meaning that the Slave servers are not guaranteed to have a current picture of the data that is in the Master.
* Many organizations use the Master/Slave approach for high-availability as well, but it suffers from this same limitation given that the Slave servers are not necessarily current with the Master. If a catastrophic failure of the Master server occurs, any transactions that are pending for replication will be lost, a situation that is highly unacceptable for most business transaction applications

## 服务高可用

## 分布式一致性算法

**分布式一致算法需要考虑场景。**
> * 一般计算机系统做状态管理或者多副本，假设是系统组件的行为不是任意的，通俗的说，你可以宕机可以断网可以延时，但是不能撒谎。这种场景下主要是Paxos及其变种，Raft经常被单拿出来是因为它的形式相对更容易理解。从软件角度说，etcd, zookeeper(**Zab**)，都是采用此类一致性算法。

## CAP
### 概念
> * [参考](http://www.hollischuang.com/archives/666)
* Consistency
   * all nodes see the same data at the same time
   * 从客户端来看，一致性主要指的是多并发访问时更新过的数据如何获取的问题
   * 从服务端来看，则是更新如何复制分布到整个系统，以保证数据最终一致
* Availability
 * Reads and writes always succeed  **服务一直可用，而且是正常响应时间**
 * 对于一个可用性的分布式系统，每一个非故障的节点必须对每一个请求作出响应
* Partition Tolerance分区容错性 
 * the system continues to operate despite arbitrary message loss or failure of part of the system

### 场景理解
{% asset_img 27.png %}
> **假设在N1和N2之间网络断开的时候，有用户向N1发送数据更新请求，那N1中的数据V0将被更新为V1，由于网络是断开的，所以分布式系统同步操作M，所以N2中的数据依旧是V0；这个时候，有用户向N2发送数据读取请求，由于数据还没有进行同步，应用程序没办法立即给用户返回最新的数据V1，怎么办呢？有二种选择，第一，牺牲数据一致性，响应旧的数据V0给用户；第二，牺牲可用性，阻塞等待，直到网络连接恢复，数据更新操作M完成之后，再给用户响应最新的数据V1。**


### 实际
> * CA without P：如果不要求P（不允许分区），则C（强一致性）和A（可用性）是可以保证的。但其实分区不是你想不想的问题，而是始终会存在，因此CA的系统更多的是允许分区后各子系统依然保持CA
* CP without A：如果不要求A（可用），相当于每个请求都需要在Server之间强一致，而P（分区）会导致同步时间无限延长，如此CP也是可以保证的。很多传统的数据库分布式事务都属于这种模式
* AP wihtout C：要高可用并允许分区，则需放弃一致性。一旦分区发生，节点之间可能会失去联系，为了高可用，每个节点只能用本地数据提供服务，而这样会导致全局数据的不一致性。现在众多的NoSQL都属于此类



## 分布式事务


### 转账的例子
> 
* 原则
  * 大事务 = 小事务（原子事务）+ 异步（消息通知）
  * BASE模型 Basically Available、Soft State、Eventually Consistent
* 流程
  * [时序图文章](http://aoyouzi.iteye.com/blog/2392406)
  * A账户扣钱，往消息队列发送一条记录(本地事务，两个动作确保原子性)
  * B账户收到MQ消息，往账户加钱
  * 用定时系统轮询消息队列的方式(即订阅和系统轮询两种方式结合)，**处理B没收到消息的情况**
  * 消息里应有一个标志位，标识消息是否被处理过，防止B账户加两遍钱，其他业务可以考虑是否将消费方的业务设置成幂等的
  * 消息出列后，消费者对应的业务操作要执行成功。如果业务执行失败，消息不能失效或者丢失。需要保证消息与业务操作一致，**主流的MQ产品都具有持久化消息的功能。如果消费者宕机或者消费失败，都可以执行重试机制的（有些MQ可以自定义重试次数）**

### 下单扣库存先后顺序问题
> 
* 单库直接用数据库事务
* 分库但在同一数据库分片里仍旧可以使用事务
* 跨库的情况只能是最终一致性模型

## 隔离术
### Hystrix
> * 熔断
* 降级的fallback
 * fallback代码在客户端，作用在客户端调用服务时，远程服务不可用，则调用fallback，取老数据
* 资源(线程池)隔离
{% asset_img 12.png %}


## 负载均衡

### 分类
> 
* 全局的运营商+区域层面的负载均衡，主要功能是就近原则的调度
* 机房或集群内部的负载均衡，主要实现流量均摊、合理利用资源等
* [阿里中间件团队文章](http://jm.taobao.org/2016/06/02/zhangwensong-and-load-balance/)

### DNS实现LB节点水平扩展和就近访问(GSLB)
> 
* 基于DNS
* 大型网站总是部分使用DNS域名解析，利用域名解析作为第一级负载均衡手段，解析得到的一组服务器不是实际提供web服务的物理服务器，而是**同样提供负载均衡服务的内部服务器(比方nginx)**，这组服务器再进行负载均衡，将请求分发到真实的web服务器上
* [LB节点实现水平扩展和就近访问](https://buluo.qq.com/p/detail.html?bid=11729&pid=8386231-1495136170)

### 四层LB vs 七层LB
> * 四层LB的特点一般是在网络和网络传输层(TCP/IP)做负载均衡，而七层则是指在应用层做负载均衡
* 四层LB对于应用侵入比较小，这一层的LB对应用的感知较少，同时应用接入基本不需要针对LB做任何的代码改造
* 七层负载均衡一般对应用本身的感知比较多，可以结合一些通用的业务流量负载逻辑和容灾逻辑做成很细致的负载均衡和流量导向方案，但是一般接入时，应用需要配合做相应的改造
* 互联网时代流量就是钱啊，对于流量的调度的细致程度往往是四层LB难以满足的
**LVS工作在第4层，抗压能力强，并发小的时候用nginx足够**

### 在服务端做还是在客户端做
> 
* 服务端做的好处是客户端职责单一，缺点是每次要到负载均衡的服务中心取服务地址，多了一次网络请求
* 中央的代理节点自身的处理速度和网卡带宽的瓶颈，以及LB自身的高可用
* 客户端做则需要在本地维护一套注册中心的server列表，并时刻同步信息

## 分布式协调服务框架
### eureka实现服务注册发现
> * 添加@EnableDiscoveryClient注解后，项目就具有了服务注册的功能，这里的CLient可以理解成此服务对于eureka来说是一个客户端，注册在其上面
* @EnableFeignClients：启用feign进行远程调用
* feign调用

```plain
    @FeignClient(name= "spring-cloud-producer")
    public interface HelloRemote {
        @RequestMapping(value = "/hello")
        public String hello(@RequestParam(value = "name") String name);
    }
```

### etcd、Zookeeper和Consul等实现服务发现、注册的对比
> * 与Zookeeper和etcd不一样，Consul内嵌实现了服务发现系统，所以这样就不需要构建自己的系统或使用第三方系统，[参考文章](http://dockone.io/article/667)
* springcloud的Eureka
* [关于各框架实现的参考](https://luyiisme.github.io/2017/04/22/spring-cloud-service-discovery-products/)

{% asset_img 11.png %}

### HTTP动态负载均衡
* [参考](http://xiaorui.cc/2016/10/16/nginx%E5%8A%A8%E6%80%81%E9%85%8D%E7%BD%AE%E5%8F%8A%E6%9C%8D%E5%8A%A1%E5%8F%91%E7%8E%B0%E9%82%A3%E4%BA%9B%E4%BA%8B/)
* dns动态解析，非常适合依赖域名来辨识主机的集群服务，多用在类aws、阿里云这样的云服务
* 模板渲染
* 服务发现动态配置重载.   

> * Consul + Consul-template(每次去Consul拉数据往模板里填)实现动态负载均衡(增加减少实例不用手动改nginx.conf的upstream)，缺点是每次每次配置变化(upstream 服务加入、退出)都会由系统自动执行nginx的reload功能，有一定的损耗
 * {% asset_img 29.png %}
* 不reload就实现动态变更upstream
 * tengine的dyups模块
 * 微博的[upsync](https://github.com/weibocom/nginx-upsync-module)
 * openResty的balancer_by_lua

## 分布式环境下session的处理
> 
* 基于cookie
 * 安全性，大小限制
* session sticky
 * 同一个用户的请求都路由到同一机子
* session replication
 * 在每台服务器上都放一份session，机子多时问题大
* session集中管理
 * 基于容器的第三方的容器拓展，比方Tomcat、jetty，问题是应用绑死了容器
 * 自定义，可以引入redis这样的nosql数据库
 * spring-session和shiro这样的解决方案

## 海量数据去重、个别元素是否在数组中
### BitMap
#### 应用场景
> * 没有重复元素的整数进行排序
* 找出某个数是否在海量数据中出现过

#### 原理
> * 如果海量数据是数组，比方10亿个元素，int[10亿]会占太多内存，但如果可以用1bit来表示一个整数是否存在，则可以节省32倍的空间。bit[10亿/32]，由于java中不存在bit这样的数据类型，所以要把每个数映射到数组int[i]的某一个二进制位上，**用位运算符来进行处理**（数据插入，和查找）
* 需要的临时存储空间为 int[10亿/32+1] temp;
* 先定位某个数在temp中的第几个数，然后再定位是这个数的第几个二进制位

{% asset_img 8.jpg 假设这40亿int数据为：6,3,8,32,36,......，那么具体的BitMap表示为 %}


### 2-BitMap
> * 场景，在10亿整数中找出不重复的数
* 为每个整数分配2bit，用不同的0、1组合来标识特殊意思，如00表示此整数没有出现过，01表示出现一次，11表示出现过多次

### Bloom Filter
#### [简介](https://my.oschina.net/kiwivip/blog/133498)
> **用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难**
False Positive即误报，比方垃圾邮件、缓存服务器都是可以接受的。False Negative即漏报，Bloom Filter不会存在漏报的情况

#### 过程
> * 大小为m的bit数组，k个hash函数
* 插入某个元素，经过k个hash函数处理，得到的k个index对应的bit元素置1
* 查询某个元素是否出现过，则经过k个函数处理得到k个index，如果对应的bit元素都是1，则该元素很大可能是出现过（也可能不存在）

#### Counting Bloom Filter
##### 处理的场景
标准的Bloom Filter是一种很简单的数据结构，它只支持插入和查找两种操作。在所要表达的集合是静态集合的时候，标准Bloom Filter可以很好地工作，但是如果要表达的集合经常变动，标准Bloom Filter的弊端就显现出来了，因为它不支持删除操作


## 10亿大小的文件排序
> * 内存排序直接pass
* 利用分治策略把中间结果放在外存(好多个小文件N)，再利用多路归并排序，结果写入原文件，第一行写入min(F1,F2,……Fn)，后面以此类推

## top K问题(海量数据找出K个**出现频率最高**的数字，或**前K个最大**的元素)
### 数据的切分
> * 整数的切分有直接取模、乘积取整、平方取中(根据样本的分布情况)
* 字符串切分，MD5、JS HASH、SDBM HASH


> * **注意如果是取前K个出现频率最高的元素**，比方8个元素取前2个出现频次最多的元素，平均分组后为{6,6,5,4}和{3,2,1,4}，则如果直接在各个分组里取前2位去归并，这样的极端情况会把4给漏掉，**注意和取前K个大的数情况是不一样的**

* [ref1](http://blog.csdn.net/zyq522376829/article/details/47686867)
* [ref2](http://www.cnblogs.com/qieerbushejinshikelou/p/3961217.html)
* [ref3](http://blog.csdn.net/hackbuteer1/article/details/7622869)
* 局部淘汰法(对于数量巨大的情况，堆完全载入内存是没问题的)。维护一个K大小的数组，遍历后面N-K个元素，每次替换掉数组里的最小数，复杂度为**O(N\*K)**。如果把数组换成堆，则每次直接替换堆顶的元素再update堆，复杂度为**O(N\*log K)**
* 最小堆的性质-每个节点均不大于其左右子节点，堆顶为最小元素

```plain
public int findKthLargest(int[] nums, int k) {
  PriorityQueue<Integer> minQueue = new PriorityQueue<>(k);
  for (int num : nums) {
    if (minQueue.size() < k || num > minQueue.peek())
      minQueue.offer(num);
    if (minQueue.size() > k)
      minQueue.poll();
  }
  return minQueue.peek();
}
```


* **理论最优QuickSelect**，最快是O(n)，平均是O(N log2 N)，最差是O(N^2)。假设N个数存在数组S中，随机找一个元素X，Sa中所有元素大于X，Sb中所有元素小于X，这时有两种可能。Sa大小大于K，需要返回Sa中的前K大元素；而如果Sa大小小于K，则需返回Sa中所有元素和Sb中最大的前K-|Sa|个元素，递归求解


### 优化方法
> 可以把所有10^9个数据分组存放，比如分别放在1000个文件中。这样处理就可以分别在每个文件的10^6个数据中找出最大的10000个数，合并到一起在再找出最终的结果
 
### bloom filter
> * [网上文章](https://my.oschina.net/kiwivip/blog/133498)
* 应用场景，HTTP缓存服务器、web爬虫、垃圾邮件过滤……

<span id="dynamic proxy" />

## 代理
### why 
> 
* the **main intent of a proxy is to** control access to the target object, **rather than to** enhance the functionality of the target object. 
* 实现InvocationHandler接口
* The access control includes 
 * synchronization 
 * authentication
 * remote access (RPC)
 * lazy instantiation (Hibernate, Mybatis) 
 * AOP (transaction)

### JDK自带的，ASM，CGLIB(基于ASM包装)，JAVAASSIST 


### 代理模式和装饰者模式区别
>　　The difference between the Proxy Pattern and the DecoratorPattern is one of intent: Proxy provides access control, while Decorator adds functionality

### 图解
{% asset_img 5.png %}


## 一致性哈希

### 简介
> 一致性哈希的目的，是希望“在增加服务器的时候降低数据移动规模，让尽可能多的数据保留在原有的服务器”上。一个设计良好的分布式哈希方案应该具有良好的单调性，即服务节点的增减不会造成大量哈希重定位。一致性哈希算法就是这样一种哈希方案。

### 实现
> * 在一个环上，先确定用哈希函数H确定服务器在环上的位置，再用H确定某个用户每次请求在环上的位置，顺时间方向走，第一个遇到的服务器就是其请求该定位到的服务器

### 虚拟节点
> 用于解决在服务节点太少时，容易因为节点分部不均匀而造成数据倾斜问题。

## 数据处理框架
### storm
> * Distributed and fault-tolerant realtime computation: stream processing, continuous computation, distributed RPC, and more
* Clojure实现
* Storm是分布式的、实时数据流分析工具，数据是源源不断产生的，例如Twitter的Timeline

### spark
> * scala实现
* 超速判断、网站在线用户特别多  之类的问题，与大数据查询毫无关系。它所能处理的“大”是指 实时在线请求量的 “大”。我认为这两个东西 就不应该做对比。是不同领域的东西。它们之间快慢比较根本没有意义。引用一段话：爬犁和宝剑哪个快？要看是用来耕地，还是砍人——hadoop像个爬犁，耕地很凶悍；storm像把宝剑，砍人锋利无比。

### akka
> * scala实现

### kafka
> * sacla和java实现

### hadoop
> * Hadoop是基于Map/Reduce模型的，处理海量数据的离线分析工具

### ELK
> * 简而言之， 如果说日志是埋在土里的宝藏，那么ELK是开采宝藏的蓝翔挖掘机
* [知乎专栏](https://zhuanlan.zhihu.com/p/22400290)

{% asset_img 7.png %}

## zookeeper
### 关键技术
> * 数据结构Znode + 原语 + watcher机制
* 临时节点依赖于创建它们的session，但**对所有客户端可见，不能有子节点**
* 客户端watch某个节点仅被触发一次
* {% asset_img 25.png 传统解决单点的方案(keepalived)，如果出现网络问题没收到ack，会出现split-brain双主的问题 %}
* {% asset_img 26.png 利用znode的临时属性解决split-brain问题，zookeeper在这里是一个仲裁者的角色 %}


## keepalived
> 
* 基于VRRP(Virtual Redundent Routing Protocol)协议
 * **问题描述**: 在现实的网络环境中两台需要通信的主机大多数情况下并没有直接的物理连接。对于这样的情况它们之间路由怎样选择主机如何选定到达目的主机的下一跳路由
   * 在主机上使用动态路由协议(RIP、OSPF等)，但在主机上配置动态路由是非常不切实际的因为管理、维护成本以及是否支持等诸多问题
   * 在主机上配置静态路由，但路由器|默认网关|default gateway却经常成为单点故障
* **VRRP协议的目的就是为了解决静态路由单点故障问题；VRRP通过竞选(election)协议来动态的将路由任务交给LAN中虚拟路由器中的某台VRRP路由器**

{% asset_img 28.gif MS结构的LB节点实现简单高可用 %}

## 限流
### 维度
节点级别 vs 集群级别（redis  INCRBY， 额外多一次访问Redis的消耗）
客户端 vs 服务端
服务级别 vs 方法级别
框架本身的连接/线程限制
	tomcat nginx 
接入层限流
	nginx + lua 
	有人会纠结如果应用并发量非常大那么redis或者nginx是不是能抗得住；不过这个问题要从多方面考虑：你的流量是不是真的有这么大，是不是可以通过一致性哈希将分布式限流进行分片
### 限流算法
> 
* 参考
  * [张开涛的文章](https://mp.weixin.qq.com/s?__biz=MzIwODA4NjMwNA==&mid=2652897781&idx=1&sn=ae121ce4c3c37b7158bc9f067fa024c0#rd)
  * [网文，在张开涛文章基础上](http://www.kissyu.org/2016/08/13/%E9%99%90%E6%B5%81%E7%AE%97%E6%B3%95%E6%80%BB%E7%BB%93/)
  * [白衣江南文章](http://calvin1978.blogcn.com/articles/ratelimiter.html)
* 令牌桶算法和漏桶算法
  * 漏桶算法是均匀的流出(请求)速率
  * 令牌桶算法是允许流量一定程度的突发
  * Guava的RateLimiter实现了令牌桶算法
* 计数器和滑动窗口算法
  * 计数器就是低精度的滑动窗口算法。针对计数器算法，比方我们的设定是1min可以进100个请求，那在59秒可以进100个请求，1min的时候可以再进100个，那相当于在1秒内涌进了200个请求，系统有可能崩。
* Dubbo
  * 客户端 actives="10"参数  支持方法级的限制  ActiveLimitFilter实现
  * 服务端  executes="10"参数 ExecuteLimitFilter 实现
  * TpsLimitFilter 支持 方法级的 tps限流

## 系统设计

<span id="seize business" />

### 秒杀业务

#### 个人总结
* web层用redis UID 五秒之类的请求服务层的次数
* 

#### 注意点
> 
* [网文](https://github.com/hackjutsu/Hackjutsu/blob/master/source/_posts/%E7%A7%92%E6%9D%80%E7%B3%BB%E7%BB%9F%E6%9E%B6%E6%9E%84%E5%88%86%E6%9E%90%E4%B8%8E%E5%AE%9E%E6%88%98.md)

#### 架构
{% asset_img 19.png 实时统计可能会用到大数据框架 %}

{% asset_img 16.png 原子计数器放库存 nosql服务器集群可以扛足够多的秒杀并发量 %}

{% asset_img 17.png 地址暴露接口根据时间动态变化，不能放在CDN，要用redis服务器扛 %}

{% asset_img 18.png 秒杀动作的事务过程，可以把插明细和减库存写在存储过程里，减少行锁的时间 %}

#### 优化方向
> * 将请求尽量拦截在系统上游（不要让锁冲突落到数据库上去）
* 动静态数据分离，详情页面静态化落到CDN上
* 充分利用缓存，秒杀买票，这是一个典型的读多些少的应用场景，大部分请求是车次查询，票查询，下单和支付才是写请求

#### 备注
> 
* 秒杀的URL要动态化，不能是死的
* 秒杀按钮是否可点要在前端用js控制，因为页面是静态的，时间让js文件在客户端本地取
* 将秒杀系统独立部署，甚至使用独立域名，使其与网站完全隔离
* 各层次的优化，页面层简单的像按钮点击之后变灰，恶意刷的比方基于用户uid在一定时间内只允许透传一定量的请求
* 保证所有用户看到的库存规律一致
* 保证单个用户看到的库存规律一致

#### 参考链接
> * [58沈剑的文章](http://www.infoq.com/cn/articles/flash-deal-architecture-optimization)

## 系统问题排查

### io过高
* [参考](http://bencane.com/2012/08/06/troubleshooting-high-io-wait-in-linux/)
  1. 确认系统负载上升是否由于I/O异常导致，top看iowait数值，或者iostat -x 2 5，看util参数
  2. 找出引起高I/O的进程，```ps -eo state,pid,cmd | grep "^D";``` 查找D状态的进程，状态码的含义看ps的说明


### cpu过高的排查过程
>  
* 先通过top命令查看占用cpu高的PID
* 用ps指令展示该pid下的线程并按cpu排序
* 结合jstack指令查看该线程的信息

# 复盘
## alibaba

### 10亿请求里找出最热门的10种商品
#### 数据怎么切

### ConcurrentHashMap和Collections.synchronizedCollection方法处理的容器区别
### 线程池和ForkJoinPool区别

### java内存模型
### 垃圾回收
### 数据库事务级别

## QQ

### 线程挂起是否占用cpu时间
> * [跳至spinLock](#spinLock)
* 现在的理解是自旋锁这样的机制会占用，而像线程调用共享变量的wait函数则不占用cpu

### 微服务
> 
* 解耦
* 单独开发、部署
* 单独扩展，另外cpu密集型的任务和IO密集型的任务可以部署在更适合的机器上，比方Amazon提供的云服务是有分类的

### tcp三次握手
### 跨域脚本攻击和XSS
### netty的架构

## flyme
> 
* mq消息的到达顺序
* mq消息重复处理问题
 * 正常情况下出现重复消息的概率其实很小，如果由消息系统来实现的话，肯定会对消息系统的吞吐量和高可用有影响，应放在消费端的业务代码处理
 * 消费端处理消息的业务逻辑保持幂等性
 * 利用一张日志表来记录已经处理成功的消息的ID，如果新到的消息ID已经在日志表中，那么就不再处理这条消息
* 插入排序伪码
* mysql的mvcc
* redis数据类型
* redis持久化方案比较
* spring的prototype和singleton
* 全局负载均衡
* RxJava
* redis集群和主备设置部署

## didi
> 
* 3个外部服务，券服务、余额服务、支付服务，全都是分布式的环境下
 * 支付某张订单，优先调券服务，不够再调账户余额，最后不够再调支付服务，怎么设计？3个服务都提供了回滚接口，但不保证回滚成功。
* 滴滴司机账户里2天以内的订单收益是提不了现的，现在有张单司机走了线下支付，滴滴无法抽成，要在司机之前的订单收益里扣，尽可能牵涉少的订单
* 订单表uid、orderId、createTime、其他业务字段……
 * 要分库分表
 * 既要支持用户维度uid查询又要支持orderId订单维度查询

## 58
> 
* 给1亿用户发条信息，不要发重了
  * 布隆过滤器

## bei ke 
* dubbo的bean怎么注入  多注册中心
  * [参考](http://cxis.me/2017/03/21/Dubbo%E4%B8%AD%E6%B6%88%E8%B4%B9%E8%80%85%E5%88%9D%E5%A7%8B%E5%8C%96%E7%9A%84%E8%BF%87%E7%A8%8B%E8%A7%A3%E6%9E%90/)
* 库存结构的表设计  
  * 第一层分 大品类 解决最后一层因sku多而导致货位过多的问题
  * 推荐货位的优化问题
* 数据源怎么取  注解能否动态 
  * [stackoverflow注解实现运行时修改值 还没细看](https://stackoverflow.com/questions/24367384/modify-field-annotation-value-dynamically)
  * [注解+微型mybatis](https://juejin.im/entry/57e496fd7db2a20063a24a3a)
* 阻塞队列实现   会不会被别的程序调了notify 
   * synchronized关键字 锁类内部的 list成员变量，不要锁Queue的this
* 同一用户 入参uid  一个小时内只能登录5次 
* BeanPostProcessor BeanFactory
* 异常处理器能不能处理springmvc 抛的异常 像参数的格式
* mybatis怎么拿到数据源拿connection

## sougou
* 可重复读隔离级别怎么解决不可重复读问题 
   * 加个share lock？
* 多线程打印1 2 3 
   * object锁wait notify方法处理
* 热点问题 超买超卖问题

## qunaer
* 服务慢怎么定位
    （GC、慢sql)
* synchronized Map的迭代问题  
* 超卖  队列扛写怎么同步  
* 两个链表是否相等O方的bug  
    aa aa bb 
    bb bb aa
* dubbo服务端 限流

# shopee
tcp 半连接 比方server 发起了 finish client 回了 ack 那server不会再往client发数据，能不能读？
redis的io多路复用
coroutine
对象存活  可达性分析 **值得细化**
线程的状态及切换

# lazada
* starter作用  spring的拓展
* 回收算法
* annotation 注释的bean怎么取

# oppo2019
* nginx高可用 keepalive脑裂
* **dubbo超时重试粒度不够细的扩展**
* 分库分表扩容时怎么不停机

* dubbo 扩展多语言客户端怎么做？
* 各个版本的特性，未来扩展方向
* sharding JDBC原理 

* ScheduledThreadPoolExecutor可能存在的问题
* java的队列分类
* 线程状态 流转
* mysql的隔离级别为什么默认是RR
* spring 事务的传播级别

## 其他
* 抢红包业务设计 [跳转](#seize business)
* 编码实现死锁
* jvm优化参数
* 数据安全、MD5

### java事件委托机制
> 比观察者模式更灵活 [参考](http://blog.csdn.net/qiumuxia0921/article/details/52067604)

### nginx服务器的缓存不正确需手工清理
> * nginx商业版有purger功能
* ngx_cache_purge

# 项目准备
> 
* 解决问题
   * 架构 keepalived + tengine + service (脑裂问题、选主、智能dns、RPC、NIO、nginx不reload配置文件问题)
   * 注册流程的优化(解耦、消息的持久化、分布式锁)

# 第一轮 12-08 模糊的点
* 线程状态 + jstack
  * 为什么分waiting和blocked
  * jstack怎么分析线程状态
* 线程中断  
* hashMap 

https://www.jianshu.com/u/aa69380a24e9  爱奇艺架构师博客

application  starter包  怎么初始化那个bean


