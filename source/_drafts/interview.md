---
title: interview
tags:
---

# 基础

## 线程和进程对比
> * [leetcode](https://discuss.leetcode.com/topic/90877/process-vs-thread/2)
* [表格](http://www.differencebetween.info/difference-between-process-and-thread)

<!-- more -->

## 多线程
> Java里面进行多线程通信的主要方式就是**共享内存**的方式，共享内存主要的关注点有两个：可见性和有序性。加上复合操作的原子性，可以认为Java的线程安全性问题主要关注点有3个：**可见性、有序性和原子性。**
**Java内存模型（JMM）解决了可见性和有序性的问题，而锁解决了原子性的问题。**

### 队列
#### 阻塞队列
##### 代码实现
> * 数组 + notify、wait方法
   * [实现](http://tutorials.jenkov.com/java-concurrency/blocking-queues.html)
   ```java
   public class BlockingQueue {

     private List queue = new LinkedList();
     private int  limit = 10;
   
     public BlockingQueue(int limit){
       this.limit = limit;
     }
   
     public synchronized void enqueue(Object item) throws InterruptedException  {
       while(this.queue.size() == this.limit) {
         wait();
       }
       if(this.queue.size() == 0) {
         //唤醒那些堵塞在dequeue方法的线程
         notifyAll();
       }
       this.queue.add(item);
     }
   
   
     public synchronized Object dequeue()
     throws InterruptedException{
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
* 用juc包里的类实现

##### JDK提供
> * ArrayBlockingQueue
* LinkedBlockingQueue
* PriorityBlockingQueue(无界)
* DelayQueue(无界)

#### 非阻塞队列
> * ConcurrentLinkedQueue


## 线程池 
> * [并发编程网链接](http://ifeve.com/java-threadpool/)
* [江南白衣](http://calvin1978.blogcn.com/articles/java-threadpool.html)

### 参数
    ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, 
    TimeUnit unit, BlockingQueue<Runnable> workQueue, ThreadFactory threadFactory, 
    RejectedExecutionHandler handler)

#### workQueue 工作队列

#### RejectedExecutionHandler
> * AbortPolicy
* CallerRunsPolicy，A handler for rejected tasks that runs the rejected task directly in the calling thread of the execute method
* DiscardOldestPolicy
* DiscardPolicy

#### 处理流程
![线程池处理流程](http://ifeve.com/wp-content/uploads/2012/12/Java%E7%BA%BF%E7%A8%8B%E6%B1%A0%E4%B8%BB%E8%A6%81%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B.jpg)

**线程池-->队列**-->扩容-->拒绝策略

#### 配置线程池考虑什么
> * 任务类型，IO还是CPU，还是混合型
* 任务的依赖性：是否依赖其他系统资源，如数据库连接
* CPU密集型任务配置尽可能少的线程数量，如配置Ncpu+1个线程的线程池。IO密集型任务则由于需要等待IO操作，线程并不是一直在执行任务，则配置尽可能多的线程，如2\*Ncpu。Amdahl法则 [细节](http://ifeve.com/how-to-calculate-threadpool-size/)
* 混合型看能不能拆分
* 需要优先级的用PriorityBlockingQueue队列
* 要用有界的队列，不然有可能会因为**外部或内部**因素撑爆系统


### 线程中断
> * **Java中断机制是一种协作机制，也就是说通过中断并不能直接终止另一个线程，而需要被中断的线程自己处理中断。**
* Thread.interrupted() static方法
* Thread.isInterrupted() 测试线程是否已经中断。线程的中断状态不受该方法的影响
* Thread.interrupt() 中断线程
* **interrupt方法是唯一能将中断状态设置为true的方法。静态方法interrupted会将当前线程的中断状态清除，但这个方法的命名极不直观，很容易造成误解，需要特别注意**
* **当处理完InterruptedException后想要传播中断状态，必须要么重新抛出捕获的InterruptedException，要么通过Thread.currentThread().interrupt()重新设置中断状态**。

#### 一个实例
##### 需求
　　参考自**[文章](https://praveer09.github.io/technology/2015/12/06/understanding-thread-interruption-in-java/)**的部分代码，后面讲线程中断状态的还需再细化。
　　开个线程在控制台打印0-9九个数字，一秒打一个。线程start后，主线程休眠3秒，然后请求子线程终止，在主线程完全关闭应用之前，至多等待子线程1秒去终止。子线程需马上响应主线程的终止请求。
##### 代码
``` java
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
```java
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
### 概览
> * [IO模型](http://blog.leanote.com/post/joesay/Concurrency-Model-Part-1-IO-Concurrency)
* [Scalable IO in java](http://gee.cs.oswego.edu/dl/cpjslides/nio.pdf)

### reactor和proactor

### IO模型的种类 
> * [郭无心的回答](https://www.zhihu.com/question/27991975) 此种方式下是指应用发起一个IO操作以后，不等待内核IO操作的完成，等内核完成 IO 操作以后会通知应用程序，这其实就是**同步和异步最关键的区别，同步必须等待或者主动的去询问 IO 是否完成**
* JAVA NIO是同步非阻塞io。**同步和异步说的是消息的通知机制，阻塞非阻塞说的是线程的状态。**
* 对于同步的事件，你只能以阻塞的方式去做。而对于异步的事件，阻塞和非阻塞都是可以的
* [严肃的回答](https://www.zhihu.com/question/19732473/answer/20851256)

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
> * ChannelHandler分inbound、outbound，在ChannelPipeline中组成链条，顺序处理业务，A ChannelHandlerContext represents an association between a ChannelHandler and a ChannelPipeline and is created whenever a ChannelHandler is added to a Channel-Pipeline
* Channel : EventLoop = n : 1
* EventLoopGroup : EventLoop = n : 1
* An EventLoop is bound to a single Thread for its lifetime
* All I/O events processed by an EventLoop are handled on its dedicated Thread
* A  common  reason  for  installing  a  single ChannelHandler in multiple ChannelPipelines is to gather statistics across multiple Channels
* 面向对象表示多对多关系则需要一个中间对象，SelectionKey。selector和selectableChannel都持有这个selectionkey集合
* Boss thread: they do main work like connect, bind and pass them for real work to worker threads. 
* Worker thread: they do the real work.
* in netty3,the threading model used in previous releases guaranteed only that inbound (previously called upstream) events would be executed in the so-called i/o thread(corresponding to netty4’s eventloop).all outbound(downstream) events were handled by the calling thread,which might be the i/o thread or any other.
* netty is asynchronous and event-driven. asynchronous说的是outbound io operations，event-driven 应该对应的是io inbound operations.

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
#### 实现
> * 是个数组，但数组的每个节点是个链表
* java8中，如果某个节点的链长度超过8，会用红黑树来替代数组，处理的是由于hashCode太多冲突造成的链太长而导致的查询蜕化问题。另外，get(key)发现冲突，需要用key.equals(k)去查找对应的entry


#### [哈希表的容量一定要是2的整数次幂](http://www.cnblogs.com/peizhe123/p/5790252.html)
> * 计算key放在哪个位置时，用hash值对length取模可以确保均匀
* length为2的整数次幂的话，hashValue & (length-1)就相当于对length取模

#### 为什么要二次hash
##### 过程 
> HashMap的key hash计算时先计算hashCode(),然后进行二次hash

##### 为什么
> * 由于Key的哈希值的分布直接决定了所有数据在哈希表上的分布或者说决定了哈希冲突的可能性，因此为防止糟糕的Key的hashCode实现（例如低位都相同，只有高位不相同，与2^N-1取与后的结果都相同）
* [参考文章1](http://www.jianshu.com/p/8b372f3a195d)
* [非常detail实现的文章](http://www.jasongj.com/java/concurrenthashmap/)

#### 为什么String, Interger这样的wrapper类适合作为键
> 因为String是不可变的，也是final的，而且已经重写了equals()和hashCode()方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode()，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。

#### load factor 
#### ConcurrentHashMap和Collections.synchronizedCollection容器区别
> * ConcurrentHashMap. It allows concurrent modification of the Map from several threads **without the need to block them**. Collections.synchronizedMap(map) creates a blocking Map which will degrade performance, albeit **ensure consistency**(if used properly). Use the second option if you need to ensure data consistency, and each thread needs to have an up-to-date view of the map. Use the first if performance is critical, and each thread only inserts data to the map, with reads happening less frequently
* [stackoverflow](https://stackoverflow.com/questions/510632/whats-the-difference-between-concurrenthashmap-and-collections-synchronizedmap)


## Spring
### bean的scope
> * singleton作用域的Bean只会在每个Spring IoC容器中存在一个实例，而且其完整生命周期完全由Spring容器管理
* prototype，Spring does not manage the complete lifecycle of a prototype bean
* [代码示例](http://memorynotfound.com/spring-bean-scopes-singleton-vs-prototype/)

### BeanFactory和ApplicationContext的区别
> * [参考文章](http://www.cnblogs.com/heyongjun1997/p/5944674.html)
* 两者都是通过xml配置文件加载bean, ApplicationContext和BeanFacotry相比, 提供了更多的扩展功能
* BeanFactory是延迟加载,如果Bean的某一个属性没有注入，BeanFacotry加载后，直至第一次使用调用getBean方法才会抛出异常；而ApplicationContext则在初始化自身是检验，这样有利于检查所依赖属性是否注入

### Spring Bean provide thread safety?
>　　The default scope of Spring bean is singleton, so there will be only one instance per context. That means that all the having a class level variable that any thread can update will lead to inconsistent data. Hence in default mode spring beans are **not thread-safe**. However we can change spring bean scope to request, prototype or session to achieve thread-safety **at the cost of performance**. It’s a design decision and based on the project requirements.

### AOP 
> * [原理](http://www.cnblogs.com/zs234/p/3267623.html)
* aop可以由AspectJ这样的框架实现，在编译时对目标类进行增强
* 动态代理的实现是运行时增强
* [跳转至动态代理](#dynamic proxy)

### IOC [原理](http://www.cnblogs.com/zs234/p/3257750.html)
> * 处理解耦的问题。类A调用B，要A处理B的初始化问题，现在把B的初始化问题丢给spring容器，解耦A、B。

### spring boot
> * 传统意义上，我们开始一个项目整个开发环境就要搭建一个很长的时间，期间如果加入更多的特性时还要更麻烦，spring boot把大量的常见组件都组合了起来，比如 hibernate jdbc mongodb jmx等等很多很多，**只要引入相关的组件基本上就是零配置就可以用了**。
* **比较适合微服务部署方式，不再是把一堆应用放到一个Web服务器下，重启Web服务器会影响到其他应用，而是每个应用独立使用一个Web服务器，重启动和更新都很容易**。

### spring cloud
> * 从技术架构上降低了对大型系统构建的要求，使我们以非常低的成本（技术或者硬件）搭建一套高效、分布式、容错的平台
* 是一系列框架的集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册(Eureka)、配置中心、消息总线、负载均衡、断路器(Hystrix)、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。

## CopyOnWrite容器
### why 
> **不用锁**

### 应用场景
> 读多写少的场景

### 缺点
> * 内存占用问题，由于每次都拷贝原内容，如果容器很大则会占很多内存
* 数据一致性问题，只能保证最终一致性，而不是实时的一致性

## JVM

### 参数
> [江南白衣](http://calvin1978.blogcn.com/articles/jvmoption-2.html)

#### GC参数
> * -XX:+PrintGCDetails 
* -Xloggc:logs/gc.log

#### heap相关参数
> * -Xms20M heap最小 
* -Xmx20M heap最大 
* -Xmn20M 年轻代大小(1.4or lator) （eden+ 2 survivor space)

### jvm内存结构
{% asset_img 2.png 在Sun JDK中，本地方法栈和Java栈是同一个%}

{% asset_img 20.png 注意heap区的划分 %}

<span id="Class Loader" />
#### Class Loader
> 分类
* BootStrap Class Loader 负责加载rt.jar文件中所有的Java类，即Java的核心类都是由该ClassLoader加载
* Extension Class Loader 负责加载一些扩展功能的jar包
* System Class Loader 负责加载启动参数中指定的Classpath中的jar包及目录，通常我们自己写的Java类也是由该ClassLoader加载
* User Defined Class Loader

> 工作过程
* 装载 将Java二进制代码导入jvm中，生成Class文件
* 链接 a）校验：检查载入Class文件数据的正确性 b）准备：给类的静态变量分配存储空间 c）解析：将符号引用转成直接引用 
* 初始化 对类的静态变量，静态方法和静态代码块执行初始化工作

##### 双亲委派模型(Parents Delegation Model)
> * **双亲委派模型要求除了顶层的启动类加载器之外，其余的类加载器都应当有自己的父类加载器**
* [双亲委派机制是为了安全而设计的](http://www.cnblogs.com/lanxuezaipiao/p/4138511.html)，如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有当父加载器反馈自己无法完全这个加载请求时，子加载器才会尝试自己去加载
{% asset_img 9.png %}

#### Java Stack
#### Method Area
#### Heap
> Java堆是被所有线程共享的一块内存区域，主要用于存放对象实例
{% asset_img 3.jpg 永久代空间在Java SE8特性中已经被移除%}
{% asset_img 4.jpg %}

#### java内存分配策略
> * 局部变量、形参都是从栈中分配空间
* **栈分配空间时，如为一个即将要调用的程序模块分配数据区时，应该事先知道这个数据区的大小。也就是说虽然分配是在程序运行时进行的，但是分配的大小是确定的、不变的，而这个大小是在编译期决定的而不是运行时**
* 堆在应用程序运行时请求操作系统给自己分配内存，由于操作系统管理内存分配，所以效率较栈要低。但优点在于编译器不需要知道从堆那里分配多少存储空间，也不必知道存储的数据要在堆里停留多长的时间，有更大的灵活性
* 像多态这样的机制，所需的存储空间只有在运行时创建了对象之后才能确定，必须是堆内存的分配

### 类的加载机制
[跳转](#Class Loader)


### GC算法 垃圾回收
> * 对象存活判断
* GC算法 标记 -清除算法、复制算法、标记-压缩算法，我们常用的垃圾回收器一般都采用分代收集算法
* 垃圾回收器
* [面向GC的JAVA编程](http://blog.hesey.net/2014/05/gc-oriented-java-programming.html)

#### 如何判断对象是否存活
> * 引用计数法，实现简单，判定高效，但不能解决对象之间相互引用的问题
* 可达性分析法

### GC分析 命令调优
> * GC日志分析
* 调优命令
* 调优工具

#### 日志分析
> * [参考文章](http://swcdxd.iteye.com/blog/1859858)
* 5.617: [GC 5.617: [ParNew: 43296K->7006K(47808K), 0.0136826 secs] 44992K->8702K(252608K), 0.0137904 secs] [Times: user=0.03 sys=0.00, real=0.02 secs]  
* 5.617（时间戳）: [GC（Young GC） 5.617（时间戳）: 
[ParNew（使用ParNew作为年轻代的垃圾回收期）: 43296K（年轻代垃圾回收前的大小）->7006K（年轻代垃圾回收以后的大小）(47808K)（年轻代的总大小）, 0.0136826 secs（回收时间）] 
44992K（堆区垃圾回收前的大小）->8702K（堆区垃圾回收后的大小）(252608K)（堆区总大小）, 0.0137904 secs（回收时间）] 
[Times: user=0.03（Young GC用户耗时） sys=0.00（Young GC系统耗时）, real=0.02 secs（Young GC实际耗时）]  


## JMM(java memory model)
> * [jenkov](http://tutorials.jenkov.com/java-concurrency/java-memory-model.html)
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
> * 被修饰的变量存于main memory，而不是CPU cache，避免编译器优化带来的问题(比方instructions reordered)
* 线程1写了volatile变量X，线程2可以读到最新的值，同时**线程1在此(写X)之前对其他非volatile变量的写操作比方有非volatile变量Y，也会被写回到main memory中，即线程2只要做了读X的动作，读到的Y也是最新的Y** (有点像一些数据库中间件，线程对主库做了写操作，则后续的读操作会被强制路由到主库，而不是从库，因为有可能因为主从复制延时的问题读不到最新值)

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

#### ReadWriteLock
> * 读写锁的理念是，只要没有任何线程写入变量，并发读取可变变量通常是安全的。所以读锁可以同时被多个线程持有，只要没有线程持有写锁。
* 如果读锁先被T1获取了，则T2要获取写锁也要等
* 如果写锁先被T1获取了，同时T2，T3要获取读锁，则T1释放写锁之后T2，T3可以同时获取读锁

#### StampedLock（和optimistic lock有关）

### 和Semaphore对比
> 信号量可以维护整体的准入许可。这在一些不同场景下，例如你需要限制你程序某个部分的并发访问总数时非常实用

<span id="spinLock"/>
### 自旋锁(spinLock)
#### 为什么需要自旋锁
　　互斥锁的问题是，线程休眠和唤醒是相当昂贵的操作，如果mutex仅被锁住很短一段时间，比使线程休眠和唤醒的时间还要更短，互斥锁并不高效在这样的场景下

#### 自旋锁的问题
* 如果自旋锁被持有的时间过长， 其它尝试获取自旋锁的线程会一直轮训自旋锁的状态， 这将非常浪费CPU的执行时间
* 单CPU系统上使用自旋锁，一个线程持有了便持续**占用CPU**，其他线程没有机会获得锁

#### 操作系统层面Hybird SpinLocks，Hybrid Mutexes
[并发编程网文章](http://ifeve.com/practice-of-using-spinlock-instead-of-mutex/)

### AbstractQueuedSynchronizer(关键类)

### 分布式锁
<span id="distributed lock redis" />
#### 基于redis的实现
> * [并发编程网的参考](http://ifeve.com/redis-lock/)
* [魏子珺的分析](http://weizijun.cn/2016/03/17/%E8%81%8A%E4%B8%80%E8%81%8A%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E7%9A%84%E8%AE%BE%E8%AE%A1/)

#### 关于数据库、zookeeper和redis实现分布式锁的对比
> * 数据库实现是把对锁的竞争丢给了数据库，存在问题包括比方服务挂了对锁的释放要在应用层用定时任务实现，要实现可重入的功能比较复杂，数据库宕机问题，最后是性能问题
* redis实现最大优势是性能

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

## 排序
### 归并排序
> [wiki](https://zh.wikipedia.org/wiki/%E5%BD%92%E5%B9%B6%E6%8E%92%E5%BA%8F)

## 基础编程题

## servlet
> * [工作原理](https://stackoverflow.com/questions/3106452/how-do-servlets-work-instantiation-sessions-shared-variables-and-multithreadi)
* [单例](https://stackoverflow.com/questions/8011138/servlet-seems-to-handle-multiple-concurrent-browser-requests-synchronously)

## 设计模式
> * 适配器
   * 实现一个接口类不想实现其所有方法，可以继承其抽象子类(适配器类\*\*adapter)只覆写想要覆写的方法
   * [插线口标准实例](http://blog.csdn.net/zhangjg_blog/article/details/18735243)
   ```java
   public class SocketAdapter    
        implements GermanSocketInterface {   //实现旧接口  
  
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

## linux commands 
### top
### iostat
### free
### vmstat
### strace

# 熟悉

## 索引

### 索引类型
> * normal
* unique
* full-text

### 建索引方式

#### BTree
> * 长文本字段建索引方式 
* Index selectivity概念 

#### Hash 

### clustered index(聚簇索引)
> * [参考](http://blog.csdn.net/ak913/article/details/8026743)
* 不是某种索引，而是数据组织方式，像字典的以拼音查和以偏旁部首查。

## TCP
### TIME_WAIT数量过多解决
> * {% asset_img 15.png 主动发起断链的会进入TIME_WAIT状态 %}
* 设置tcp_tw_reuse和tcp_tw_recycle参数，HTTP服务器设置keep-alive参数，让客户端断链
* **使用tcp_tw_reuse和tcp_tw_recycle来解决TIME_WAIT的问题是非常非常危险的，因为这两个参数违反了TCP协议**
* 许令波的深入分析java web技术内幕关于tcp参数调优
* [丁火笔记](https://huoding.com/2012/01/19/142)
* [丁火笔记](https://huoding.com/2013/12/31/316)
* [酷壳](http://coolshell.cn/articles/11564.html)
* [王宏江](http://hongjiang.info/nginx-tomcat-keep-alive/)

## HTTP

### http method

### 长链接
#### HTTP 1.0+ 用Keep-Alive
#### HTTP 1.1 用persistent

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

#### 客户端分片
> 分片的规则全都放在应用程序层，足够灵活和自定义

#### 代理分片
> * Twemproxy，最大的痛点在于，无法平滑地扩容/缩容
* codis

#### Redis Cluster
> 请求发到任意一个redis实例，由redis cluster判断这个请求应该到哪个实例，再回给客户端重新进行请求。类似重定向

### Sentinel保证高可用
> * [庄周梦蝶的文章](http://blog.fnil.net/blog/255ccfae971d6d30b0921120d327490b/)
* [redis官方教程](http://redis.majunwei.com/topics/sentinel.html)

```
sentinel monitor mymaster 127.0.0.1 6379 2
sentinel down-after-milliseconds mymaster 60000
sentinel failover-timeout mymaster 180000
sentinel parallel-syncs mymaster 1

sentinel monitor resque 192.168.1.3 6380 4
sentinel down-after-milliseconds resque 10000
sentinel failover-timeout resque 180000
sentinel parallel-syncs resque 5
```
> * 设置了一个实例叫mymaster，另一个叫resque
* sentinel monitor <master-group-name> <ip> <port> <quorum>  
假设有5个sentinel进程，quorum在这里是2，则当有2个sentinel发现mymaster挂了的时候，其中一个会尝试发起故障转移的动作，只要当至少3个(5 / 2 + 1)sentinel认定mymaster挂了的时候，故障转移流程会真正开始

## 异步servlet


# 了解

## RPC
> * 为什么需要
* 用**动态代理**实现透明化远程服务调用(封装通信细节让用户像以本地调用方式调用远程服务)
* 对消息进行编码和解码
* 通信框架
* 发布自己的服务
 * 人肉
 * 自动，zookeeper，leader选举机制可以确保服务的高可用

## RxJava

## java8
> * stream
* lamda
* CompletableFuture

## 缓存

## 消息中间件
### ActiveMQ
#### 集群配置方式有哪些
> * MASTER/SLAVE模式
   * Pure master slave
{% asset_img 13.png %}
   * 基于共享文件系统
{% asset_img 14.png %}
   * 基于数据库
   * 基于zookeeper
* Cluster模式
 * Broker clusters（静态static）
 * network of brokers（动态multicast）
   * [network of brokers的运作机制和可能存在的无法做到高可用的问题](http://www.cnblogs.com/yjmyzz/p/activemq-ha-using-networks-of-brokers.html)
   * [network of brokers搭建集群方案](http://www.cnblogs.com/yjmyzz/p/activemq-broker-cluster.html)
   * [参考](http://blog.csdn.net/yinwenjie/article/details/51205822)

## mysql

### 事务级别
|                             | 脏读   |  不重复读(NonRepeatable read) |  幻读(Phantom Read)  |
| --------                    | :----: |  :----:                       | :----:  |
| 未提交读(read uncommitted)  | 存在   |   存在                        |  存在   |
| 已提交读(read committed)    | X      |   存在                        |  存在   |
| 可重复读(repeatable read)   | X      |   X                           |  存在   |
| 串行(serializable)          | X      |   X                           |    X    | 
> * [参考](http://www.cnblogs.com/zhoujinyi/p/3437475.html)
* 脏读，事务A读了事务B还未提交的数据
* 不可重复读，事务A在事务过程中多次读取某个数据，而此数据可能在事务B中修改过导致A多次读取的数据不一致
* 幻读，事务A对表t中所有数据进行更新但还有其他业务未做完，事务B向t插入了一条数据，如果A再去读t中的数据，会发现还有未更新的数据，这种情况即是幻读


### 数据库遇到性能瓶颈的处理顺序
> 数据库水平分片作为数据库性能瓶颈的最终解决方案，确实可以有效的解决这个问题。但是它将业务逻辑变得非常复杂（主要是**关联表冗余**和**字段冗余**，以及这些数据的更新），并且有分布式事务这个难题。所以不到必要时刻，尽量不要轻易尝试数据库分片。而是先去选择对业务影响较小的读写分离，垂直分库等方案。
[见文](http://www.scienjus.com/database-sharding-review/)

### replication
#### 实现细节
##### binary log

#### 方式
##### one-way, asynchronous replication
> * M给S写数据，一旦网络延迟或中断，则Slave和Master就会出现数据不一致；甚至如果此时Master宕机，会导致未发送到slave的数据丢失

#####  semisynchronous replication
> * 和同步相比即故障恢复时间较长

##### 同步写slave

### 高可用

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
##### 垂直
##### 水平
###### 水平分表和分片(sharding)
> * 参考
   * [聚合](http://www.cnblogs.com/maanshancss/p/5327803.html)
   * [黄钧航](http://blog.csdn.net/qingrx/article/details/8756839)
  * 水平分表是在同一库在对表进行水平，字表间仍可以通过union这样的关键字进行联合查询
  * 水平分表只能在单台机子上的单个数据库进行，如果服务器本身的性能以及达到瓶颈，则分表不会有帮助
  * 水平分库后多个库可以部署在不同机子上，充分利用多台服务器的性能

###### 水平分片的sharding key
> * Sharding扩容与系统采用的路由规则密切相关：基于散列的路由能均匀地分布数据，但却需要数据迁移，同时也无法避免对达到上限的节点不再写入新数据；基于增量区间的路由天然不存在数据迁移和向某一节点无上限写入数据的问题，但却存在“热点”困扰
* 选择
 * 基于id的hash，可以使用一致性hash
 * id简单分区间，比方1-2w在A库，2w-4w在B库
 * 基于时间分片
 * 基于地理位置

###### 关联表的冗余字段问题

查询用户 1 关注的所有男性用户，并且以每页 20 条进行分页
```
单库情况
select * 
from user u 
inner join follow f on f.follow_id = u.user_id 
where f.user_id = 1
```

```
多库的情况
ids = 
select follow_id from follow where user_id = 1 limit 20;
use db_1;
select * from user where id in (ids) amd sex = 1;
use db_2;
select * from user where id in (ids) amd sex = 1;
```

**上述查询可能出现数据量不足的情况，需要把sex字段从user表冗余到follow表中去**

###### 关联表的冗余表
> **对于表之间的关联关系，尽量将一对多，多对一中的关联数据放在同一个分片下（例如一个用户和他所发的所有微博哦），多对多关系最坏情况需要在在关联双方所在的数据库都存放一条记录，并且保持这多条记录的同步更新。**

###### 水平分片的分页查询解决方法 
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

## 分布式事务
### 转账的例子
> * A账户扣钱，往消息队列发送一条记录(本地事务，两个动作确保原子性)
* B账户采用持久化订阅队列的方式收消息，往账户加钱
* 用定时系统轮询消息队列的方式(即订阅和系统轮询两种方式结合)，处理B没收到消息的情况
* 消息里应有一个标志位，标识消息是否被处理过，防止B账户加两遍钱，其他业务可以考虑是否将消费方的业务设置成幂等的
* 消息出列后，消费者对应的业务操作要执行成功。如果业务执行失败，消息不能失效或者丢失。需要保证消息与业务操作一致，**主流的MQ产品都具有持久化消息的功能。如果消费者宕机或者消费失败，都可以执行重试机制的（有些MQ可以自定义重试次数）**

## 隔离术
### Hystrix
> * 熔断
* 降级的fallback
 * fallback代码在客户端，作用在客户端调用服务时，远程服务不可用，则调用fallback
* 资源(线程池)隔离
{% asset_img 12.png %}


## 负载均衡

### 分类
> * 链路负载均衡，比方由DNS server决定这个请求路由到哪台web服务器
* 集群负载均衡
 * 硬件
 * 软件
* 操作系统负载均衡

### 作用
> * 全局的运营商+区域层面的负载均衡，主要功能是就近原则的调度
* 机房或集群内部的负载均衡，主要实现流量均摊、合理利用资源等
* [阿里中间件团队文章](http://jm.taobao.org/2016/06/02/zhangwensong-and-load-balance/)

### global server load balance 
> * 基于DNS
* 大型网站总是部分使用DNS域名解析，利用域名解析作为第一级负载均衡手段，解析得到的一组服务器不是实际提供web服务的物理服务器，而是**同样提供负载均衡服务的内部服务器**，这组服务器再进行负载均衡，将请求分发到真实的web服务器上

### 四层LB vs 七层LB
> * 四层LB的特点一般是在网络和网络传输层(TCP/IP)做负载均衡，而七层则是指在应用层做负载均衡
* 四层LB对于应用侵入比较小，这一层的LB对应用的感知较少，同时应用接入基本不需要针对LB做任何的代码改造
* 七层负载均衡一般对应用本身的感知比较多，可以结合一些通用的业务流量负载逻辑和容灾逻辑做成很细致的负载均衡和流量导向方案，但是一般接入时，应用需要配合做相应的改造
* 互联网时代流量就是钱啊，对于流量的调度的细致程度往往是四层LB难以满足的

### 在服务端做还是在客户端做
> * 服务端做的好处是客户端职责单一，缺点是每次要到负载均衡的服务中心取服务地址，多了一次网络请求
* 客户端做则需要在本地维护一套注册中心的server列表，并时刻同步信息

## 分布式协调服务框架
### eureka实现服务注册发现
> * 添加@EnableDiscoveryClient注解后，项目就具有了服务注册的功能，这里的CLient可以理解成此服务对于eureka来说是一个客户端，注册在其上面
* @EnableFeignClients：启用feign进行远程调用
* feign调用

```java
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
> * Consul + Consul-template实现动态负载均衡，缺点是每次每次配置变化(upstream 服务加入、退出)都会由系统自动执行nginx的reload功能，有一定的损耗
* 不reload就实现动态变更upstream
 * tengine的dyups模块
 * 微博的upsync
 * openResty的balancer_by_lua

## 分布式环境下session的处理
### 基于cookie
> * 安全性，大小限制

### session sticky
> * 同一个用户的请求都路由到同一机子

### session replication
> * 在每台服务器上都放一份session，机子多时问题大

### session集中管理
> * 基于容器的第三方的容器拓展，比方Tomcat、jetty，问题是应用绑死了容器
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
用于检索一个元素是否在一个集合中。它的优点是空间效率和查询时间都远远超过一般的算法，缺点是有一定的误识别率和删除困难
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
* 利用分治策略把中间结果放在外存(好多个小文件N)，再利用多路归并排序，结果写入原文件，第一行写入min(F1,F2,……Fn)，以此类推

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
``` java
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

<span id="dynamic proxy"></span>
## 代理
### why 
>　　the **main intent of a proxy is to** control access to the target object, **rather than to** enhance the functionality of the target object. 
The access control includes 
* synchronization 
* authentication
* remote access (RPC)
* lazy instantiation (Hibernate, Mybatis) 
* AOP (transaction)

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
> 数据结构Znode + 原语 + watcher机制

#### Znode
> * 临时节点依赖于创建它们的session，但对所有客户端可见，不能有子节点
* 客户端watch某个节点仅被触发一次

#### watch
> * 分data watch和child watch

## 系统设计

<span id="seize business" />

### 秒杀业务
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
> * 将秒杀系统独立部署，甚至使用独立域名，使其与网站完全隔离
* 各层次的优化，页面层简单的像按钮点击之后变灰，恶意刷的比方基于用户uid在一定时间内只允许透传一定量的请求
* 保证所有用户看到的库存规律一致
* 保证单个用户看到的库存规律一致

#### 参考链接
> * [58沈剑的文章](http://www.infoq.com/cn/articles/flash-deal-architecture-optimization)

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
> * 解耦
* 单独开发、部署
* 单独扩展，另外cpu密集型的任务和IO密集型的任务可以部署在更适合的机器上，比方Amazon提供的云服务是有分类的

### tcp三次握手
### 跨域脚本攻击和XSS
### netty的架构

## flyme
> * mq消息的到达顺序
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

## 深圳meituan
> * 一个小商店，客户去买东西，表设计，怎么取到用户的手机号存进用户表


## 其他
### 抢红包业务设计 [跳转](#seize business)
### 编码实现死锁
### jvm优化参数
### 数据安全、MD5

### java事件委托机制
> 比观察者模式更灵活 [参考](http://blog.csdn.net/qiumuxia0921/article/details/52067604)

### nginx服务器的缓存不正确需手工清理
> * nginx商业版有purger功能
* ngx_cache_purge

# TODO
> * linux指令(结合着jvm)
* 还需从头回顾的内容
 * jvm整体(周一晚，至迟周二完成)
 * netty结构
* 原理类准备
 * zookeeper
 * spring 
 * redis
 * mybatis
* 工厂模式
* 分布式还未覆盖的地方
 * 限流量
 * 服务降级
 * 消息系统
   * 构建
   * 分布式事务
* 集合类
* SpringMVC的分发过程 [来自](http://blog.csdn.net/u013256816/article/details/51787470)
* SpringBean的加载过程
* LVS和Nigix的区别及使用场景 
* Javac的编译过程 
* Java虚拟机结构分析
* 用脚本实现加实例不reload Nginx 结合consul 全局负载均衡
* sharding回顾，读写分离
* redis主备和同步机制
* Netty聊天application
* TCP关键题相关
* HTTP
* java8 api实现sql函数，和Stream、CompletableFuture细化

# 项目准备
> * 解决问题
   * 收益汇总，在数据库层汇总(悲观锁)而不是应用层
* 负责内容


