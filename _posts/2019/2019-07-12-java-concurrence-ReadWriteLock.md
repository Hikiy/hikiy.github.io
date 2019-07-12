---
layout: post
title:  "Java并发：ReadWriteLock读写锁"
date:   2019-07-12 17:28:00 +0200
categories: 并发
excerpt: 
tagg: Java
---

# ReadWriteLock：读写锁

## 什么是读写锁

- 允许多个线程同时读共享变量
- 只允许一个线程写共享变量
- 如果一个写线程正在执行写操作，禁止读线程读共享变量

Java SDK 并发包提供了读写锁——ReadWriteLock，非常容易使用，并且性能很好。它的实现类是ReentrantReadWriteLock

## 实现一个缓存
在下面的代码中，我们声明了一个 Cache<K, V> 类，其中类型参数 K 代表缓存里 key 的类型，V 代表缓存里 value 的类型。缓存的数据保存在 Cache 类内部的 HashMap 里面，HashMap 不是线程安全的，这里我们使用读写锁 ReadWriteLock 来保证其线程安全。

```
class Cache<K,V> {
  final Map<K, V> m =
    new HashMap<>();
  final ReadWriteLock rwl =
    new ReentrantReadWriteLock();
  // 读锁
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  // 读缓存
  V get(K key) {
    r.lock();
    try { return m.get(key); }
    finally { r.unlock(); }
  }
  // 写缓存
  V put(String key, Data v) {
    w.lock();
    try { return m.put(key, v); }
    finally { w.unlock(); }
  }
}
```

### 缓存数据的初始化
如果源头数据的数据量不大，就可以采用一次性加载的方式:
![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/E2E908295ECB4492A87B43AA7EE30108/18521)

如果源头数据量非常大，那么就需要按需加载了，按需加载也叫懒加载，指的是只有当应用查询缓存，并且数据不在缓存里的时候，才触发加载源头相关数据进缓存的操作
![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/56D7CB41B1214BCB8E4620B4445DA76E/18523)

### 实现缓存的按需加载
```
class Cache<K,V> {
  final Map<K, V> m =
    new HashMap<>();
  final ReadWriteLock rwl = 
    new ReentrantReadWriteLock();
  final Lock r = rwl.readLock();
  final Lock w = rwl.writeLock();
 
  V get(K key) {
    V v = null;
    // 读缓存
    r.lock();         ①
    try {
      v = m.get(key); ②
    } finally{
      r.unlock();     ③
    }
    // 缓存中存在，返回
    if(v != null) {   ④
      return v;
    }  
    // 缓存中不存在，查询数据库
    w.lock();         ⑤
    try {
      // 再次验证
      // 其他线程可能已经查询过数据库
      v = m.get(key); ⑥
      if(v == null){  ⑦
        // 查询数据库
        v= 省略代码无数
        m.put(key, v);
      }
    } finally{
      w.unlock();
    }
    return v; 
  }
}
```

在获取写锁之后，我们并没有直接去查询数据库，而是在代码⑥⑦处，重新验证了一次缓存中是否存在，原因是在高并发的场景下，有可能会有多线程竞争写锁。假设缓存是空的，没有缓存任何东西，如果此时有三个线程 T1、T2 和 T3 同时调用 get() 方法，并且参数 key 也是相同的。那么它们会同时执行到代码⑤处，但此时只有一个线程能够获得写锁，假设是线程 T1，线程 T1 获取写锁之后查询数据库并更新缓存，最终释放写锁。此时线程 T2 和 T3 会再有一个线程能够获取写锁，假设是 T2，如果不采用再次验证的方式，此时 T2 会再次查询数据库。T2 释放写锁之后，T3 也会再次查询一次数据库。

## 锁的升级和降级

### 升级
上面按需加载的示例代码中，在①处获取读锁，在③处释放读锁，那是否可以在②处的下面增加验证缓存并更新缓存的逻辑呢？详细的代码如下。
```
// 读缓存
r.lock();         ①
try {
  v = m.get(key); ②
  if (v == null) {
    w.lock();
    try {
      // 再次验证并更新缓存
      // 省略详细代码
    } finally{
      w.unlock();
    }
  }
} finally{
  r.unlock();     ③
}
```
先是获取读锁，然后再升级为写锁，对此还有个专业的名字，叫锁的升级。但是在ReadWriteLock面前当然不行啦，读锁没释放，无法获取写锁的，然后就永久等待了

### 降级
ReadWriteLock不能升级锁，但是却可以降级锁。

就是在持有写锁的时候，能够再持有读锁。最后释放写锁的时候，仍然持有读锁。这样的好处应该就是能保证写了之后还能继续读吧。

```
class CachedData {
  Object data;
  volatile boolean cacheValid;
  final ReadWriteLock rwl =
    new ReentrantReadWriteLock();
  // 读锁  
  final Lock r = rwl.readLock();
  // 写锁
  final Lock w = rwl.writeLock();
  
  void processCachedData() {
    // 获取读锁
    r.lock();
    if (!cacheValid) {
      // 释放读锁，因为不允许读锁的升级
      r.unlock();
      // 获取写锁
      w.lock();
      try {
        // 再次检查状态  
        if (!cacheValid) {
          data = ...
          cacheValid = true;
        }
        // 释放写锁前，降级为读锁
        // 降级是可以的
        r.lock(); ①
      } finally {
        // 释放写锁
        w.unlock(); 
      }
    }
    // 此处仍然持有读锁
    try {use(data);} 
    finally {r.unlock();}
  }
}

```
## 其它
### ReentrantLock 支持公平模式和非公平模式

### ReentrantLock 的读锁和写锁都实现了Lock接口

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.12  
> 更新日期：2019.07.12
