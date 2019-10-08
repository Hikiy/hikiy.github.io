---
layout: post
title:  "Java并发：CountDownLatch和CyclicBarrier"
date:   2019-07-15 16:41:00 +0200
categories: 并发
excerpt: 解决一些线程等待问题
tagg: Java
---

# CountDownLatch和CyclicBarrier

在多线程下，如果有个线程A需要等待另外两个线程都完成才完成的话，要怎么实现呢？

如果该线程A执行的时候，希望另外两个线程也继续并行执行下一个操作，要怎么实现呢？

比如下面的对账系统:

![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/FEBD0B06A4CD48CBB3CBD2D3C6A843B6/18612)

## 第一个问题

### 使用线程的join()方法
```
while(存在未对账订单){
  // 查询未对账订单
  Thread T1 = new Thread(()->{
    pos = getPOrders();
  });
  T1.start();
  // 查询派送单
  Thread T2 = new Thread(()->{
    dos = getDOrders();
  });
  T2.start();
  // 等待 T1、T2 结束
  T1.join();
  T2.join();
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
} 
```

但是，JDK已经帮我们实现了。

### 使用CountDownLatch实现
while 循环里面每次都会创建新的线程，而创建线程可是个耗时的操作。所以最好是创建出来的线程能够循环利用，所以用线程池解决会更好。

**但是使用线程池将不知道两个线程什么时候执行完毕**

可以加一个计数器，初始值位2，两个线程，每个执行完减1。计数器为0的时候执行对账操作。

**CountDownLatch**就实现了这个功能

主要使用**latch.await()** 来实现对计数器等于 0 的等待
```
// 创建 2 个线程的线程池
Executor executor = 
  Executors.newFixedThreadPool(2);
while(存在未对账订单){
  // 计数器初始化为 2
  CountDownLatch latch = 
    new CountDownLatch(2);
  // 查询未对账订单
  executor.execute(()-> {
    pos = getPOrders();
    latch.countDown();
  });
  // 查询派送单
  executor.execute(()-> {
    dos = getDOrders();
    latch.countDown();
  });
  
  // 等待两个查询操作结束
  latch.await();
  
  // 执行对账操作
  diff = check(pos, dos);
  // 差异写入差异库
  save(diff);
}
```

这样就很好地解决了第一个问题

## 第二个问题

### 计数器和唤醒

利用双队列、计数器实现

- 添加计数器
- 线程执行完将结果存入队列，并减计数器
- 对账线程从队列拿数据，执行完后，唤醒另外两个线程。
- 将计数器置回2

JDK也帮我们实现了

### 使用CyclicBarrier实现

创建一个计数器初始值为 2 的 **CyclicBarrier**，CyclicBarrier 的时候，我们还传入了一个回调函数，调用**await()**方法减计数器，当计数器减到 0 的时候，会调用这个**回调函数**。

**CyclicBarrier 的计数器有自动重置的功能，当减到 0 的时候，会自动重置你设置的初始值**。

```
// 订单队列
Vector<P> pos;
// 派送单队列
Vector<D> dos;
// 执行回调的线程池 
Executor executor = Executors.newFixedThreadPool(1);
final CyclicBarrier barrier =
  new CyclicBarrier(2, ()->{
    executor.execute(()->check());
  });
  
void check(){
  P p = pos.remove(0);
  D d = dos.remove(0);
  // 执行对账操作
  diff = check(p, d);
  // 差异写入差异库
  save(diff);
}
  
void checkAll(){
  // 循环查询订单库
  Thread T1 = new Thread(()->{
    while(存在未对账订单){
      // 查询订单库
      pos.add(getPOrders());
      // 等待
      barrier.await();
    }
  }
  T1.start();  
  // 循环查询运单库
  Thread T2 = new Thread(()->{
    while(存在未对账订单){
      // 查询运单库
      dos.add(getDOrders());
      // 等待
      barrier.await();
    }
  }
  T2.start();
}
```

**回调函数不能直接是方法！** CyclicBarrier是同步调用回调函数之后才唤醒等待的线程，如果我们在回调函数里直接调用check()方法，那就意味着在执行check()的时候，是不能同时执行getPOrders()和getDOrders()的

## 线程池的 Future 特性
可以利用 Future 特性来实现线程之间的等待

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.15  
> 更新日期：2019.07.15
