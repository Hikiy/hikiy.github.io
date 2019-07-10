---
layout: post
title:  "Java并发：互斥锁"
date:   2019-07-10 16:20:00 +0200
categories: 并发
excerpt: 
tagg: Java
---

# 互斥锁

## 原子性问题
原子性问题的源头是线程切换

**同一时刻只有一个线程执行**这个条件非常重要，我们称之为**互斥**。如果我们能够保证对共享变量的修改是互斥的，那么，无论是单核 CPU 还是多核 CPU，就都能保证原子性了。

## 简易锁模型

```
    加锁操作：lock()
    /**
    /* 临界区
    **/
    解锁操作：unlock()
```
但是**我们锁的是什么？我们保护的又是什么？**

## 改进后的锁模型

```
    创建保护资源R的锁：LR   
    加锁操作：lock()        
    /**                 
    /* 临界区               
    /* 受保护资源R      
    **/
    解锁操作：unlock()
```

- 把临界区要保护的资源标注出来
- 为要保护的资源它创建一把锁 LR
- 针对这把锁 LR，我们还需在进出临界区时添上加锁操作和解锁操作

## synchronized

Java 语言提供的 synchronized 关键字，就是锁的一种实现。

### 示例
```
class X {
  // 修饰非静态方法
  synchronized void foo() {
    // 临界区
  }
  // 修饰静态方法
  synchronized static void bar() {
    // 临界区
  }
  // 修饰代码块
  Object obj = new Object()；
  void baz() {
    synchronized(obj) {
      // 临界区
    }
  }
} 
```

### 规则
- 对于普通同步方法，锁是当前实例对象
- 对于静态同步方法，锁是当前类的Class对象

```
class X {
  // 修饰静态方法
  synchronized(X.class) static void bar() {
    // 临界区
  }
}

```

- 对于同步方法块，锁是synchronized括号里配置的对象

```
class X {
  // 修饰非静态方法
  synchronized(this) void foo() {
    // 临界区
  }
}
```

### 解决count +=1 问题

```
class SafeCalc {
  long value = 0L;
  synchronized long get() {
    return value;
  }
  synchronized void addOne() {
    value += 1;
  }
}
```

```
    创建保护资源R的锁：this   
    加锁操作：synchronized(this)        
    /**                 
    /* 临界区get()               临界区addOne()
    /* 受保护资源value           受保护资源value
    **/
    解锁操作：synchronized(this)     
```

## 锁和受保护资源的关系
合理的关系是:**受保护资源和锁之间的关联关系是 N:1 的关系**。也就是说，在并发中，多个受保护的资源需要共用一个锁。如果用两个锁保护同一个资源将会是不安全的。例如：

```
class SafeCalc {
  static long value = 0L;
  synchronized long get() {
    return value;
  }
  synchronized static void addOne() {
    value += 1;
  }
}
```

- get方法锁的是示例对象this
- addOne方法锁的是类本身SafeCalc.class

由于是两把锁，所以临界区没有互斥关系，这时候addOne方法对get方法没有可见性保证，就会出现问题。

## 加锁的本质

对象被创建在堆中，对象在内存中的存储布局方式分三个区域：
- 对象头
- 实例数据
- 对齐填充

在对象头中由自身运行的数据比如：锁的状态、线程持有的锁等等

sync锁的对象monitor指针指向一个**ObjectMonitor**对象，每个对象实例都会有一个 monitor。其中monitor可以与对象一起创建、销毁；亦或者当线程试图获取对象锁时自动生成。

```
objectMonitor(){
    _count = 0;
    _owner = NULL;
    _WaitSet = NULL;
    _WaitSetLock = 0;
    _EntryList = NULL;
}
```

- `_WaitSet` 用来保存每个等待锁的线程对象。
- `_owner` 指向持有的ObjectMonitor对象的线程（也就是持有锁的线程）
- `_EntryList` 当多个线程同时访问时，将线程存放到这里

### 过程
- 当线程获取到对象的monitor时，就会把`_owner`变量设置为当前线程。同时`_count`变量+1。
- 如果线程调用wait() 方法，就会释放当前持有的monitor，那么`_owner`变量就会被置为null，同时`_count`减1，并且该线程进入`_WaitSet`集合中，等待下一次被唤醒。
- 当前线程顺利执行完方法，也将释放monitor。

**所以如果锁的对象是new object 即新对象，那么加锁将无效**


### synchronized非公平锁
有一次面试就问了我synchronized是公平锁还是非公平锁，为什么？当时真的懵了。
公平的意思是先到先得。synchronized属于**非公平锁**

所有线程加入monitor的entrylist里面，去cas抢锁，更改count加1拿锁，执行完代码，释放锁count减1，和aqs机制差不多，只是所有线程不阻塞，cas抢锁，没有队列，属于非公平锁。

## 保护没有关联的多个资源

比如银行的账户密码和余额。分别创建final对象作为锁进行保护。

**不能用可变对象做锁！！**

```
class Account {
  // 锁：保护账户余额
  private final Object balLock
    = new Object();
  // 账户余额  
  private Integer balance;
  // 锁：保护账户密码
  private final Object pwLock
    = new Object();
  // 账户密码
  private String password;
 
  // 取款
  void withdraw(Integer amt) {
    synchronized(balLock) {
      if (this.balance > amt){
        this.balance -= amt;
      }
    }
  } 
  // 查看余额
  Integer getBalance() {
    synchronized(balLock) {
      return balance;
    }
  }
 
  // 更改密码
  void updatePassword(String pw){
    synchronized(pwLock) {
      this.password = pw;
    }
  } 
  // 查看密码
  String getPassword() {
    synchronized(pwLock) {
      return password;
    }
  }
}
```

## 保护有关联关系的多个资源

例如A向B转账100元，A和B的余额都需要进行保护，如果还用上面的方法进行加锁的话，A锁的是A.this。B锁的是B.this。这样无法保护。

### 方法一：持有类的锁（不推荐）
锁住Account.class类就可以实现。但是如果这么做，所有的转账都将串行，极度影响效率。
```
class Account {
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    synchronized(Account.class) {
      if (this.balance > amt) {
        this.balance -= amt;
        target.balance += amt;
      }
    }
  } 
}

```

### 方法二：使用两把锁（会引起死锁）

也就是转账的时候锁A账本的同时，去尝试锁B账本

```
class Account {
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    // 锁定转出账户
    synchronized(this) {              
      // 锁定转入账户
      synchronized(target) {           
        if (this.balance > amt) {
          this.balance -= amt;
          target.balance += amt;
        }
      }
    }
  } 
}
```
**但是会死锁！！**

比如A想转账给B，B也想转账给A，A拿了自己的锁A，B拿了自己的锁B，这时候A等B的锁阻塞，B等A的锁阻塞，永远等待！

### 方法三：破坏占用且等待（不推荐）
再转账的时候，一次性申请两个账户即可

创建一个对象用来管理临界区，这个对象必须是账户类Account.class里的单例，用这个对象来申请两个账户（不推荐）。但是这个方法仍然效率不高，因为单例还是串行，只是锁的范围缩小了，只锁住与这个转账操作相关的两个对象，一定程度上优化了。

```
class Allocator {
  private List<Object> als =
    new ArrayList<>();
  // 一次性申请所有资源
  synchronized boolean apply(
    Object from, Object to){
    if(als.contains(from) ||
         als.contains(to)){
      return false;  
    } else {
      als.add(from);
      als.add(to);  
    }
    return true;
  }
  // 归还资源
  synchronized void free(
    Object from, Object to){
    als.remove(from);
    als.remove(to);
  }
}
 
class Account {
  // actr 应该为单例
  private Allocator actr;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    // 一次性申请转出账户和转入账户，直到成功
    while(!actr.apply(this, target))
      ；
    try{
      // 锁定转出账户
      synchronized(this){              
        // 锁定转入账户
        synchronized(target){           
          if (this.balance > amt){
            this.balance -= amt;
            target.balance += amt;
          }
        }
      }
    } finally {
      actr.free(this, target)
    }
  } 
}
```
### 优化方法三：用 synchronized 实现等待 - 通知机制

**等待 - 通知机制：线程首先获取互斥锁，当线程要求的条件不满足时，释放互斥锁，进入等待状态；当要求的条件满足时，通知等待的线程，重新获取互斥锁**

Java 语言内置的 synchronized 配合wait()、notify()、notifyAll()这三个方法就能轻松实现。

同一时刻，只允许一个线程进入 synchronized 保护的临界区（这个临界区可以看作大夫的诊室），当有一个线程进入临界区后，其他线程就只能进入等待队列里等待。**这个等待队列和互斥锁是一对一的关系，每个互斥锁都有自己独立的等待队列。**

线程在进入等待队列的同时，会释放持有的互斥锁，线程释放锁后，其他线程就有机会获得锁，并进入临界区了。

**如果 synchronized 锁定的是 this，那么对应的一定是 this.wait()、this.notify()、this.notifyAll()；如果 synchronized 锁定的是 target，那么对应的一定是 target.wait()、target.notify()、target.notifyAll() 。**

```
class Allocator {
  private List<Object> als;
  // 一次性申请所有资源
  synchronized void apply(
    Object from, Object to){
    // 经典写法
    while(als.contains(from) ||
         als.contains(to)){
      try{
        wait();
      }catch(Exception e){
      }   
    } 
    als.add(from);
    als.add(to);  
  }
  // 归还资源
  synchronized void free(
    Object from, Object to){
    als.remove(from);
    als.remove(to);
    notifyAll();
  }
}
```

- 使用while是因为**notify() 只能保证在通知时间点，条件是满足的**。
- notify() 是会随机地通知等待队列中的一个线程，而 notifyAll() 会通知等待队列中的所有线程。从感觉上来讲，应该是 notify() 更好一些，因为即便通知所有线程，也只有一个线程能够进入临界区。但那所谓的感觉往往都蕴藏着风险，实际上使用 notify() 也很有风险，它的风险在于可能导致某些线程永远不会被通知到。
> 假设我们有资源 A、B、C、D，线程 1 申请到了 AB，线程 2 申请到了 CD，此时线程 3 申请 AB，会进入等待队列（AB 分配给线程 1，线程 3 要求的条件不满足），线程 4 申请 CD 也会进入等待队列。我们再假设之后线程 1 归还了资源 AB，如果使用 notify() 来通知等待队列中的线程，有可能被通知的是线程 4，但线程 4 申请的是 CD，所以此时线程 4 还是会继续等待，而真正该唤醒的线程 3 就再也没有机会被唤醒了。

所以除非经过深思熟虑，否则尽量使用 notifyAll()。

**为什么不用sleep()来挂起线程?**

wait与sleep区别在于：
- wait会释放所有锁而sleep不会释放锁资源.
- wait只能在同步方法和同步块中使用，而sleep任何地方都可以.
- wait无需捕捉异常，而sleep需要.
- sleep是Thread的方法，而wait是Object类的方法；
- sleep方法调用的时候必须指定时间



### 方法四：破坏循环等待条件
需要对资源进行排序，然后按序申请资源。这个实现非常简单，我们假设每个账户都有不同的属性 id，这个 id 可以作为排序字段，申请的时候，我们可以按照从小到大的顺序来申请。比如下面代码中，①~⑥处的代码对转出账户（this）和转入账户（target）排序，然后按照序号从小到大的顺序锁定账户。这样就不存在“循环”等待了。

```
class Account {
  private int id;
  private int balance;
  // 转账
  void transfer(Account target, int amt){
    Account left = this        ①
    Account right = target;    ②
    if (this.id > target.id) { ③
      left = target;           ④
      right = this;            ⑤
    }                          ⑥
    // 锁定序号小的账户
    synchronized(left){
      // 锁定序号大的账户
      synchronized(right){ 
        if (this.balance > amt){
          this.balance -= amt;
          target.balance += amt;
        }
      }
    }
  } 
}
```

## 死锁条件
以下**四个条件都满足**的时候发生死锁
- 互斥，共享资源 X 和 Y 只能被一个线程占用；
- 占有且等待，线程 T1 已经取得共享资源 X，在等待共享资源 Y 的时候，不释放共享资源 X；
- 不可抢占，其他线程不能强行抢占线程 T1 占有的资源；
- 循环等待，线程 T1 等待线程 T2 占有的资源，线程 T2 等待线程 T1 占有的资源，就是循环等待。

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.10  
> 更新日期：2019.07.10
