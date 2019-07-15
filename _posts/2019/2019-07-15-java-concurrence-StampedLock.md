---
layout: post
title:  "Java并发：StampedLock"
date:   2019-07-15 15:09:00 +0200
categories: 并发
excerpt: 与ReadWriteLock类似，但是一些情况下性能偏高一些
tagg: Java
---

# StampedLock 

与ReadWriteLock类似，但是一些情况下性能偏高一些

## 支持三种锁模式

- 写锁
- 悲观读锁
- 乐观读

写锁、悲观读锁的语义和 ReadWriteLock 的写锁、读锁的语义非常类似，允许多个线程同时获取悲观读锁，但是只允许一个线程获取写锁，写锁和悲观读锁是互斥的。不同的是：StampedLock 里的写锁和悲观读锁加锁成功之后，都会返回一个 stamp；然后解锁的时候，需要传入这个 stamp。相关的示例代码如下。

```
final StampedLock sl = new StampedLock();
  
// 获取 / 释放悲观读锁示意代码
long stamp = sl.readLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockRead(stamp);
}
 
// 获取 / 释放写锁示意代码
long stamp = sl.writeLock();
try {
  // 省略业务相关代码
} finally {
  sl.unlockWrite(stamp);
}
```

 StampedLock 支持乐观读的方式。ReadWriteLock 支持多个线程同时读，但是当多个线程同时读的时候，所有的写操作会被阻塞；而 StampedLock 提供的乐观读，是允许一个线程获取写锁的，也就是说不是所有的写操作都被阻塞。

 通过调用 **tryOptimisticRead()** 获取了一个 stamp，这里的 tryOptimisticRead() 就是我们前面提到的乐观读。之后将共享变量 x 和 y 读入方法的局部变量中，不过需要注意的是，由于 tryOptimisticRead() 是无锁的，所以共享变量 x 和 y 读入方法局部变量时，x 和 y 有可能被其他线程修改了。因此最后读完之后，还需要再次验证一下是否存在写操作，这个验证操作是通过调用 **validate(stamp)** 来实现的。

 ```
 class Point {
  private int x, y;
  final StampedLock sl = new StampedLock();
  // 计算到原点的距离  
  int distanceFromOrigin() {
    // 乐观读
    long stamp = sl.tryOptimisticRead();
    // 读入局部变量，
    // 读的过程数据可能被修改
    int curX = x, curY = y;
    // 判断执行读操作期间，
    // 是否存在写操作，如果存在，
    // 则 sl.validate 返回 false
    if (!sl.validate(stamp)){
      // 升级为悲观读锁
      stamp = sl.readLock();
      try {
        curX = x;
        curY = y;
      } finally {
        // 释放悲观读锁
        sl.unlockRead(stamp);
      }
    }
    return Math.sqrt(
      curX * curX + curY * curY);
  }
}
 ```

## 注意事项

### StampedLock 不支持重入

### 不要随意打断线程
如果线程阻塞在 StampedLock 的 readLock() 或者 writeLock() 上时，此时调用该阻塞线程的 interrupt() 方法，会导致 CPU 飙升。

使用 StampedLock 一定不要调用中断操作，**如果需要支持中断功能**，一定使用可中断的悲观读锁 **readLockInterruptibly()** 和写锁 **writeLockInterruptibly()**

### 记得释放每一个锁

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.15  
> 更新日期：2019.07.15
