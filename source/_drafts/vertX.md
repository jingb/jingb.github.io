---
title: vertX
tags:
---


# 计划
> 
* 实现一遍Blueprint TODO应用
* 在Blueprint Micro Shop应用的基础上，加一个服务

# 参考
> 
* [sczyh30系列](http://www.sczyh30.com/tags/Vert-x/)
* [官网金融系统的教程](http://escoffier.me/vertx-hol)

<!-- more -->
# 关键地方
> 
* 在函数式编程中，Future 实际上是一种 Monad。可以简单地把它看作是一个可以进行变换(map)和组合(compose)的包装对象。我们把这种特性叫做 monadic
* vert.x会出现callback，这个时候要解决它，reactivex和coroutine两种方式，一开始先学会用future，这是第二个坎，这个坎不是那么容易过去的，很多人学不会future，future只是java语法，一样玩死一堆人，然后思维转换一下，搞reactivex，然后接触高阶函数，这个时候顺便把脚本比如groovy之类的脚本都给收拾掉吧，这是第二个坎
* 
````java
void doAsync(A a, B b, Handler<R> handler);
// 这两种实现等价
Future<R> doAsync(A a, B b);
````

# Monad

## 参考
* [白木城](https://zhuanlan.zhihu.com/p/31863235)

## 什么是Monad，解决了什么
1. monad解决什么问题 -> 封装了副作用
2. 副作用是什么 -> 多次调用test函数，同一个输入可能出现不同的输出 
```java
int temp = 0;
public int test(int s) {
    return s + temp;
}
``` 
3. 大部分时候写出不带副作用的函数不难，**但有时候副作用不可避免，比方读文件，可能存在文件不存等问题**
  * Java通过声明异常的机制解决此类问题
  * 纯粹的函数式编程语言，e.g. Haskell，Lisp等来说，它不存在这个机制
  * Vert.x中，大多数异步API都是通过传入Lambda参数的形式实现的。**所以此时Java也会出现类似问题，传入Lambda并不困难，难在这个Lambda执行过程中，出现了异常，那该怎么办？一般是抛不到调用者的环境中去的，因为Lambda可能会在另外一个线程中被执行**
4. 回到1，Monad解决了什么问题
5. 除了Monad还有什么方式可以解决副作用的问题 -> 协程

