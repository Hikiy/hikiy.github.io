---
layout: post
title:  "Java并发：使用Semaphore实现限流器"
date:   2019-07-12 15:12:00 +0200
categories: 并发
excerpt: 
tagg: Java
---

# 使用Semaphore实现限流器

Semaphore，现在普遍翻译为“信号量”

## 信号量模型

**一个计数器，一个等待队列，三个方法。**
![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/51BCE1D4944F46C982B98C24AF089BC6/18477)

- init()：设置计数器的初始值。
- down()：计数器的值减 1；如果减1后的计数器的值小于 0，则当前线程将被阻塞，否则当前线程可以继续执行。
- up()：计数器的值加 1；如果加1后的计数器的值小于或者等于 0，则唤醒等待队列中的一个线程，并将其从等待队列中移除。

**在 Java SDK 并发包里，down() 和 up() 对应的则是 acquire() 和 release()。**

## 实现限流器
信号量可以实现互斥锁，但不仅仅于此

**Semaphore 可以允许多个线程访问一个临界区**。例如连接池、对象池、线程池等等。当然，每个连接在被释放前，是不允许其他线程使用的。

### 例子
所谓对象池呢，指的是一次性创建出 N 个对象，之后所有的线程重复利用这 N 个对象，当然对象在被释放前，也是不允许其他线程使用的。对象池，可以用 List 保存实例对象，这个很简单。但关键是限流器的设计，这里的限流，指的是不允许多于 N 个线程同时进入临界区。

我们把计数器的值设置成对象池里对象的个数 N，就能完美解决对象池的限流问题了。下面就是对象池的示例代码。

```
class ObjPool<T, R> {
  final List<T> pool;
  // 用信号量实现限流器
  final Semaphore sem;
  // 构造函数
  ObjPool(int size, T t){
    pool = new Vector<T>(){};
    for(int i=0; i<size; i++){
      pool.add(t);
    }
    sem = new Semaphore(size);
  }
  // 利用对象池的对象，调用 func
  R exec(Function<T,R> func) {
    T t = null;
    sem.acquire();
    try {
      t = pool.remove(0);
      return func.apply(t);
    } finally {
      pool.add(t);
      sem.release();
    }
  }
}
// 创建对象池
ObjPool<Long, String> pool = 
  new ObjPool<Long, String>(10, 2);
// 通过对象池获取 t，之后执行  
pool.exec(t -> {
    System.out.println(t);
    return t.toString();
});

```

**PS:对象保存在了 Vector 中，因为信号量支持多个线程进入临界区，执行list的add和remove方法时可能是多线程并发执行**

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.12  
> 更新日期：2019.07.12
