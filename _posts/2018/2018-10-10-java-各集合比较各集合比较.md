---
layout: post
title:  "Java知识：各集合比较"
date:   2018-10-10 16:00:00 +0200
categories: java
excerpt: 
tagg: Java
---

# 各集合比较

## TreeSet和TreeMap的区别与联系

TreeSet里面的绝大部分方法都是直接调用TreeMap方法来实现的。

相同点：

- 1.TreeMap和TreeSet都是有序的集合，也就是说它们存储的值都是排好序的；
- 2.TreeMap和TreeSet都是非同步集合，因此它们不能在多线程之间共享；
- 3.运行速度都比Hash集合慢，它们内部对元素操作的时间复杂度为O(logN)，而HashMap和HashSet则为O(1)。

不同点：

- 1.最主要的区别就是TreeSet实现了Set接口，而TreeMap实现了Map接口；
- 2.TreeSet只存储一个对象，而TreeMap存储两个对象Key和Value（仅仅Key对象有序）；
- 3.TreeSet中不能有重复的对象，元素是唯一的，TreeMap中的Key不能重复，而Value可以重复。

## HashSet、HashMap和HashTable的区别与联系

区别：

- 1.HashSet继承自AbstractSet类，实现了Set接口，HashMap继承自AbstractMap类，实现了Map接口，HashTable继承自Dictionary类，实现了Map接口；
- 2.HashTable中的方法是Synchronize（线程安全）的，而HashMap中的方法是非Synchronize（线程不安全）的，因为HashSet中的方法都是HashMap实现的，所以也是线程不安全的；
- 3.HashMap是Key-Value的形式存储对象，Key不能重复，但可以为null，但这样的键只能有一个；而HashSet只存储一个对象并且不能有重复的对象，可以有一个null值；HashTable不允许null值（键与值均不能为空）；
- 4.HashMap和HashSet初始大小为16，HashTable初始大小为11；
- 5.内存扩容方式：HashTable采用的是2*old+1，而HashMap和HashSet都是2*old；
- 6.哈希值的计算方法不同，Hashtable直接使用的是对象的hashCode,而HashMap则是在对象的hashCode的基础上还进行了一些变化（先计算hashcode,然后再求其在哈希表的相应位置）。

联系：

- 1.HashSet底层方法是用HashMap来实现的，所以很多性质都相同：都是线程不安全的，Key都不能重复，初始大小都为16，扩容方式都是2*old，hashCode的计算方式也相同；
- 2.HashTable和HashMap都是以Key-Value的形式存储对象；
- 3.HashTable和HashMap都实现了Map接口。

## ArrayList、LinkedList、Vector的区别与联系

区别：

- 1.线程安全

> ArrayList和LinkedList是非Synchronize（线程不安全）的而Vector是线程安全的，所以如果多线程情况下可以使用Vector；

- 2.查询、插入速度

> **ArrayList**和**Vector**采用数组方式存储数据，所以数据的**插入慢，查询快**；**LinkedList**采用**双向链表**存储数据，查询时需要按序号索引向前或向后遍历，速度慢，而插入数据只需记录本项前后项即可，插入数据快；

- 3.初始容量和扩容

> **ArrayList**的默认初始容量为10（可自定义），增长为原来的**1.5倍**（java1.7之前的为oldCapacity1.5+1，java1.7之后为1.5倍）；  
> **LinkedList**默认初始容量为1，每次增长的容量为1；  
> **Vector**默认初始容量为10，每次增长为原来的**2倍**；

联系：

- 1.都是实现了List接口；
- 2.ArrayList和LinkedList是线程不安全的；
- 3.ArrayList和Vector都是采用数组方式存储数据；

<br /><br /><br /><br />
> github: https://github.com/Hikiy  
> 作者：Hiki  
> 创建日期：2018.10.10  
> 更新日期：2018.10.10
