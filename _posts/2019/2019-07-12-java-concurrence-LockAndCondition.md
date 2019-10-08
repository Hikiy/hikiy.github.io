---
layout: post
title:  "Java并发：Lock和Condition"
date:   2019-07-12 11:50:00 +0200
categories: 并发
excerpt: 
tagg: Java
---

# Lock和Condition

为什么有synchronized还要SDK里的一些并发包？

**因为有些情况synchronized无法很好解决**，比如synchronized无法破坏不可抢占条件

## 破坏不可抢占条件三种方法
- 能够响应中断：发生死锁的时候，线程阻塞，如果线程能够响应中断，释放锁就可以了
- 支持超时
- 非阻塞地获取锁：如果获取锁失败则直接返回错误，不阻塞的话也可以破坏不可抢占的条件

Lock接口中就有这三个方案:
```
// 支持中断的 API
void lockInterruptibly() throws InterruptedException;

// 支持超时的 API
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;

// 支持非阻塞获取锁的 API
boolean tryLock();
```

## SDK中锁的大致原理


**利用了 volatile 相关的 Happens-Before 规则**

ReentrantLock，内部持有一个 volatile 的成员变量 state，获取锁的时候，会读写 state 的值；解锁的时候，也会读写 state 的值（简化后的代码如下面所示）。也就是说，在执行 value+=1 之前，程序先读写了一次 volatile 变量 state，在执行 value+=1 之后，又读写了一次 volatile 变量 state。根据相关的 Happens-Before 规则：

- 顺序性规则：对于线程 T1，value+=1 Happens-Before 释放锁的操作 unlock()；
- volatile 变量规则：由于 state = 1 会先读取 state，所以线程 T1 的 unlock() 操作 Happens-Before 线程 T2 的 lock() 操作；
- 传递性规则：线程 T2 的 lock() 操作 Happens-Before 线程 T1 的 value+=1 。
```
class SampleLock {
  volatile int state;
  // 加锁
  lock() {
    // 省略代码无数
    state = 1;
  }
  // 解锁
  unlock() {
    // 省略代码无数
    state = 0;
  }
}
```

关于Java SDK 并发包里锁和条件变量是如何实现的，可以参考《Java 并发编程的艺术》一书的第 5 章《Java 中的锁》

## 可重入锁ReentrantLock

**线程可以重复获取同一把锁**

## 构造公平锁和非公平锁
ReentrantLock 这个类有两个构造函数，一个是无参构造函数，一个是传入 **fair 参数**的构造函数。fair 参数代表的是锁的公平策略，如果传入 true 就表示需要构造一个公平锁，反之则表示要构造一个非公平锁。

```
// 无参构造函数：默认非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
// 根据公平策略参数创建锁
public ReentrantLock(boolean fair){
    sync = fair ? new FairSync() 
                : new NonfairSync();
}
```

## 用锁的最佳实践
Doug Lea《Java 并发编程：设计原则与模式》一书中，推荐的三个用锁的最佳实践，它们分别是：

- 永远只在**更新对象的成员变量**时加锁
- 永远只在**访问可变的成员变量**时加锁
- 永远不在**调用其他对象的方法**时加锁

## Lock 和 Condition 实现的管程
**线程等待和通知需要调用 await()、signal()、signalAll()**，它们的语义和 wait()、notify()、notifyAll() 是相同的。但是不一样的是，Lock&Condition 实现的管程里只能使用前面的 await()、signal()、signalAll()，而后面的 wait()、notify()、notifyAll() 只有在 synchronized 实现的管程里才能使用。如果一不小心在 Lock&Condition 实现的管程里调用了 wait()、notify()、notifyAll()，那程序可就彻底玩儿完了。

```
public class BlockedQueue<T>{
  final Lock lock =
    new ReentrantLock();
  // 条件变量：队列不满  
  final Condition notFull =
    lock.newCondition();
  // 条件变量：队列不空  
  final Condition notEmpty =
    lock.newCondition();
 
  // 入队
  void enq(T x) {
    lock.lock();
    try {
      while (队列已满){
        // 等待队列不满
        notFull.await();
      }  
      // 省略入队操作...
      // 入队后, 通知可出队
      notEmpty.signal();
    }finally {
      lock.unlock();
    }
  }
  // 出队
  void deq(){
    lock.lock();
    try {
      while (队列已空){
        // 等待队列不空
        notEmpty.await();
      }  
      // 省略出队操作...
      // 出队后，通知可入队
      notFull.signal();
    }finally {
      lock.unlock();
    }  
  }
}
```

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.12  
> 更新日期：2019.07.12
