---
layout: post
title:  "Java并发：并发容器"
date:   2019-07-16 10:40:00 +0200
categories: 并发
excerpt: 并发容器是经常会用到的，特别是List、Map、Set、Queue
tagg: Java
---

# 并发容器

并发容器是经常会用到的，特别是List、Map、Set、Queue

## List

### CopyOnWriteArrayList

**CopyOnWriteArrayList 仅适用于写操作非常少的场景，而且能够容忍读写的短暂不一致**

写的时候会将共享变量新复制一份，好处是读操作无锁。

执行写操作：
- 将array复制一份副本
- 在副本上进行写操作
- 操作完后将array指向副本

![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/51DA4F8BC7964CFA9DA5380CA1822D74/18691)

## Map

### ConcurrentHashMap
ConcurrentHashMap的key无序

### ConcurrentSkipListMap
ConcurrentSkipListMap的key有序

SkipList本身是一种数据结构“跳表”,跳表插入、删除、查询操作平均的时间复杂度是 O(log n)，理论上和并发线程数没有关系，所以
在并发度非常高的情况下，若你对 ConcurrentHashMap 的性能还不满意，可以尝试一下
ConcurrentSkipListMap。
### 共同点
key和value都不能为空，否则抛出NullPointerEeception错误

![](https://note.youdao.com/yws/public/resource/aab7147570c5edbb8c0c3eda4018495b/xmlnote/2ED68E2DA8894EF0B8336C4165A31239/18707)

## Set

### CopyOnWriteArraySet 和 ConcurrentSkipListSet

原理和前面的ConpyOnWriteArrayList、ConccurentSkipListMap的类似

## Queue

Java并发包中的Queue分两个维度来分类

- **阻塞、非阻塞**：队列满时，入队阻塞。队列空时，出队阻塞即阻塞
- **单端、双端**：单（队尾入队，队首出队），双（队首队尾都可入队出队）

组合起来就有四个分类了

阻塞用Blocking标记，单端用Queue，双端用Deque

实际工作中，一般都不建议使用无界的队列，因为数据量大了之后很容易导致 OOM。上
面我们提到的这些 Queue 中，只有 ArrayBlockingQueue 和 LinkedBlockingQueue 是支持有界的，所以在使用其他无界队列时，一定要充分考虑是否存在导致 OOM 的隐患

### 单端阻塞队列

- ArrayBlockingQueue：数组(有界)
- LinkedBlockingQueue：链表（有界）
- SynchronousQueue：不持有队列
- LinkedTransferQueue：融合 LinkedBlockingQueue 和 SynchronousQueue功能，性能比LinkedBlockingQueue更好
- PriorityBlockingQueue：支持按照优先级出列
- DelayQueue：支持延时出队。

### 双端阻塞队列

LinkedBlockingDequeue

### 单端非阻塞队列
ConcurrentLinkedQueue

### 双端非阻塞队列
ConcurrentLinkedDeque

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2019.07.16  
> 更新日期：2019.07.16
