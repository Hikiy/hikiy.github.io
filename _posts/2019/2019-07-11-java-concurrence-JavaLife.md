---
layout: post
title:  "Java并发：线程生命周期"
date:   2019-07-11 15:05:00 +0200
categories: 并发
excerpt: 状态转换、interrupt()
tagg: Java
---

# 线程生命周期

## 线程生命周期：五态
- 初始状态
- 可运行状态
- 运行状态
- 休眠状态
- 终止状态

## Java的线程声明周期：四态

- NEW（初始化状态）
- RUNNABLE（可运行 / 运行状态）
- 休眠状态
【BLOCKED（阻塞状态）
WAITING（无时限等待）
TIMED_WAITING（有时限等待）】
- TERMINATED（终止状态）

### RUNNABLE 与 BLOCKED 的状态转换
只有一种场景会触发这种转换，就是线程等待 synchronized 的隐式锁。synchronized 修饰的方法、代码块同一时刻只允许一个线程执行，其他线程只能等待，这种情况下，等待的线程就会从 RUNNABLE 转换到 BLOCKED 状态。而当等待的线程获得 synchronized 隐式锁时，就又会从 BLOCKED 转换到 RUNNABLE 状态。

**线程调用阻塞式 API 时，是否会转换到 BLOCKED 状态呢？**

**操作系统层面**，线程是会转换到休眠状态的

**JVM 层面**，Java 线程的状态不会发生变化，也就是说 Java 线程的状态会依然保持 RUNNABLE 状态。**JVM **层面并不关心操作系统调度相关的状态****

而我们平时所谓的 Java 在调用阻塞式 API 时，线程会阻塞，指的是操作系统线程的状态，并不是 Java 线程的状态。

### RUNNABLE 与 WAITING 的状态转换

第一种场景，获得 synchronized 隐式锁的线程，调用无参数的 **Object.wait()** 方法。其中，wait() 方法我们在上一篇讲解管程的时候已经深入介绍过了，这里就不再赘述。

第二种场景，调用无参数的 **Thread.join()** 方法。其中的 join() 是一种线程同步方法，例如有一个线程对象 thread A，当调用 A.join() 的时候，执行这条语句的线程会等待 thread A 执行完，而等待中的这个线程，其状态会从 RUNNABLE 转换到 WAITING。当线程 thread A 执行完，原来等待它的线程又会从 WAITING 状态转换到 RUNNABLE。

第三种场景，调用 **LockSupport.park()** 方法。其中的 LockSupport 对象，也许你有点陌生，其实 Java 并发包中的锁，都是基于它实现的。调用 LockSupport.park() 方法，当前线程会阻塞，线程的状态会从 RUNNABLE 转换到 WAITING。调用 LockSupport.unpark(Thread thread) 可唤醒目标线程，目标线程的状态又会从 WAITING 状态转换到 RUNNABLE。

### RUNNABLE 与 TIMED_WAITING 的状态转换
五种：
- 调用带超时参数的 Thread.sleep(long millis) 方法；
- 获得 synchronized 隐式锁的线程，调用带超时参数的 Object.wait(long timeout) 方法；
- 调用带超时参数的 Thread.join(long millis) 方法；
- 调用带超时参数的 LockSupport.parkNanos(Object blocker, long deadline) 方法；
- 调用带超时参数的 LockSupport.parkUntil(long deadline) 方法。

### NEW 与 RUNNABLE 状态转换
调用线程的start()方法

### 从 RUNNABLE 到 TERMINATED 状态
线程执行完 run() 方法后，会自动转换到 TERMINATED 状态，当然如果执行 run() 方法的时候异常抛出，也会导致线程终止。

**手动终止线程时使用interrupt()，不要使用stop()**

####  stop() 和 interrupt() 方法的主要区别
stop()方法终止线程不会给线程一丝机会，如果持有隐式锁也不会释放，这样其他线程永远没机会获得锁。类似的方法还有 suspend() 和 resume() 方法也不建议使用

interrupt()方法仅仅是通知线程。线程可以作一些后续操作，也可以无视通知。线程收到通知的方式分两种。**异常**和**主动检测**

**异常方法**：当线程 A 处于 WAITING、TIMED_WAITING 状态时，如果其他线程调用线程 A 的 interrupt() 方法，会使线程 A 返回到 RUNNABLE 状态，同时线程 A 的代码会触发 InterruptedException 异常。上面我们提到转换到 WAITING、TIMED_WAITING 状态的触发条件，都是调用了类似 wait()、join()、sleep() 这样的方法，我们看这些方法的签名，发现都会 throws InterruptedException 这个异常。这个异常的触发条件就是：其他线程调用了该线程的 interrupt() 方法。

当线程 A 处于 RUNNABLE 状态时，并且阻塞在 java.nio.channels.InterruptibleChannel 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 会触发 java.nio.channels.ClosedByInterruptException 这个异常；而阻塞在 java.nio.channels.Selector 上时，如果其他线程调用线程 A 的 interrupt() 方法，线程 A 的 java.nio.channels.Selector 会立即返回。

**主动检测**：如果线程处于 RUNNABLE 状态，并且没有阻塞在某个 I/O 操作上，例如中断计算圆周率的线程 A，这时就得依赖线程 A 主动检测中断状态了。如果其他线程调用线程 A 的 interrupt() 方法，那么线程 A 可以通过 isInterrupted() 方法，检测是不是自己被中断了。

**注意**：如果在sleep()过程中被打断，InterruptedException异常将会被try catch块获取。所以需要再catch块进行主动打断当前线程，不然将无法打断。例如：
```
Thread th = Thread.currentThread();
while(true) {
  if(th.isInterrupted()) {
    break;
  }
  // 省略业务代码无数
  try {
    Thread.sleep(100);
  }catch (InterruptedException e){
    Thread.currentThread().interrupt();
    e.printStackTrace();
  }
}
```

## 诊断多线程BUG

可以通过jstack命令或者Java VisulVM可视化工具将所有线程栈信息导出来。完整的线程栈信息不仅包括线程的当前状态、调用栈，还包括了锁的信息。

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.11  
> 更新日期：2019.07.11
