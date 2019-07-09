---
layout: post
title:  "Java并发：内存模型"
date:   2019-07-09 17:15:00 +0200
categories: 并发
excerpt: 
tagg: Java
---

# 内存模型

## 什么是内存模型
可见性的原因是缓存，导致有序性的原因是编译优化，那解决可见性、有序性最直接的办法就是禁用缓存和编译优化，但是这样问题虽然解决了，我们程序的性能可就堪忧了。

合理的方案应该是**按需禁用缓存以及编译优化**。

Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括 **volatile**、**synchronized** 和 **final** 三个关键字，以及七项 **Happens-Before** 规则和它的特性

## volatile
早在C语言里就有volatile了，最原始的意义就是禁用CPU缓存。也就是必须从内存中读取或者写入

下面的代码中，如果Java在低于 1.5 版本上运行，x 可能是 42，也有可能是 0；如果在 1.5 以上的版本上运行，x 就是等于 42。

```
class VolatileExample {
  int x = 0;
  volatile boolean v = false;
  public void writer() {
    x = 42;
    v = true;
  }
  public void reader() {
    if (v == true) {
      // 这里 x 会是多少呢？
    }
  }
}
```
Java 内存模型在 1.5 版本对 volatile 语义进行了增强。怎么增强的呢？答案是一项 Happens-Before 规则。

## Happens-Before规则
意思是 **前面一个操作的结果对后续操作是可见的**

和程序员相关的规则一共有如下六项，都是关于可见性的。

### 1.程序的顺序性规则
也就是一个线程中，按照程序顺序执行操作,例如下面代码，先执行`x = 42`再执行`v = true`

```
x = 42;
v = true;
```

### 2.volatile变量规则
volatile变量的写操作 Happens-Before于这个变量的读操作

### 3.管程锁定规则
一个锁的**解锁** Happens-Before 后续对这个锁的**加锁**

**管程**是一种通用的同步原语，在 Java 中指的就是 synchronized，synchronized 是 Java 里对管程的实现。

```
synchronized (this) { // 此处自动加锁
  // x 是共享变量, 初始值 =10
  if (this.x < 12) {
    this.x = 12; 
  }  
} // 此处自动解锁
```

### 4.线程启动规则
线程A启动线程B，线程B能够看到线程A再启动线程B前的操作

### 5.线程终止规则
这个规则关于线程等待，比如线程A启动线程B，然后调用线程B的join()则，线程B的所有操作都 Happens-Before 于join()操作

### 6.线程中断规则

对线程interrupt()方法的调用先行发生于被中断线程的代码检测到中断事件的发生，可以通过Thread.interrupted()方法检测到是否有中断发生。

### 7.对象终结规则

一个对象的初始化完成(构造函数执行结束)先行发生于它的finalize()方法的开始。
```
Thread B = new Thread(()->{
  // 此处对共享变量 var 修改
  var = 66;
});
// 例如此处对共享变量修改，
// 则这个修改结果对线程 B 可见
// 主线程启动子线程
B.start();
B.join()
// 子线程所有对共享变量的修改
// 在主线程调用 B.join() 之后皆可见
// 此例中，var==66
```

### 传递性
A Happens-Before B,B Happens-Before C,则A Happens-Before C

所以按照传递性，之前代码的执行顺序：
- x = 42 
- v = true (写变量)
- 读变量v
- 读变量x

## final变量
final 修饰变量时，初衷是告诉编译器：这个变量生而不变，可以可劲儿优化。在1.5版本前，final会有"逸出"的问题。

## 基本原则
- 原子性
- 可见性
- 有序性。

## 底层实现

主要是通过内存屏障(memory barrier)禁止重排序的，即时编译器根据具体的底层体系架构，将这些内存屏障替换成具体的 CPU 指令。对于编译器而言，内存屏障将限制它所能做的重排序优化。而对于处理器而言，内存屏障将会导致缓存的刷新操作。比如，对于volatile，编译器将在volatile字段的读写操作前后各插入一些内存屏障。

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.09  
> 更新日期：2019.07.09
